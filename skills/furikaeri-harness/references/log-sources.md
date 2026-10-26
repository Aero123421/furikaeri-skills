# Where agent session logs live

Checked against public docs and tools as of October 2026. Formats change between versions: always inspect a few records before writing a parser, and treat any field below as a hint, not a contract.

## Counting rules (all harnesses)

1. **Count each API response once.** Some logs write one response as several records (one per content block). Deduplicate by the response's id before adding anything.
2. **Unknown is not zero.** Never turn a missing or null field into 0. If you add up only the values you have, label the result a subtotal and give the number of responses or sessions left out.
3. **Cumulative counters need a baseline.** When a log stores running totals, usage in a period is *total at period end − total just before period start*, per session. If there is no snapshot before the start and the session didn't start inside the period, that session's period usage is unknown. Re-reading the same snapshot must not change the total.
4. **Session total ≠ period usage.** Say which one you report. A session that crosses midnight belongs partly to each day.
5. **Don't guess formats.** If ids, timestamps or fields don't look as described here, record that under Coverage instead of improvising.

Period bounds are easiest in UTC. With GNU `date` (on macOS install coreutils and use `gdate`):

```bash
START=$(date -u -d '2026-10-26 00:00 +0900' +%FT%TZ)   # 2026-10-25T15:00:00Z
END=$(date -u -d '2026-10-27 00:00 +0900' +%FT%TZ)     # 2026-10-26T15:00:00Z
```

## Claude Code

- **Path**: `~/.claude/projects/<encoded-cwd>/<session-id>.jsonl`, where the working directory has `/` and `.` replaced by `-` (e.g. `/home/me/app` → `-home-me-app`). Subagent transcripts are records with `isSidechain: true` and, in newer versions, separate files under `<session-id>/subagents/`.
- **Retention**: deleted after `cleanupPeriodDays` (default 30) — older periods may simply be gone.
- **Records**: one JSON object per line with `type` (`user`, `assistant`, `system`, `attachment`, …), `timestamp`, `sessionId`, `cwd`, `gitBranch`, `version`.
- **Model**: `message.model` on assistant records. Sessions can mix models (hand-offs, `/model`).
- **One response, several records**: each content block (thinking, text, each tool call) is its own assistant record, all sharing `message.id` and an identical `message.usage`. Count each `message.id` once — otherwise usage is multiplied by the number of blocks.
- **Usage**: `message.usage` → `input_tokens` (uncached only), `cache_creation_input_tokens`, `cache_read_input_tokens`. These are accurate once deduplicated.
- **Output tokens**: the `output_tokens` on these records is the placeholder the API reported when the response *started*, not the final count. Report output as unknown unless the log has a final figure (for example the `result` record of `claude -p --output-format stream-json`).
- **Missing usage**: relayed or handed-off turns can carry null fields or all-zero usage. Those responses are unknown, not zero.
- **Cache hit** = `cache_read / (input + cache_creation + cache_read)`, over responses with known usage.
- **Tools**: `tool_use` blocks in assistant `message.content` (`name`, `input`); results arrive in the next user record (`toolUseResult`). Skills show up as `Skill` tool calls; subagents as `Agent` (older: `Task`).
- **Compaction**: `system` records with `subtype: "compact_boundary"`, followed by a summary user message.

```bash
F=~/.claude/projects/<dir>/<session>.jsonl
jq -c '{type, subtype}' "$F" | sort | uniq -c                       # record mix
jq -r 'select(.type=="assistant") | .message.model' "$F" | sort | uniq -c
jq -r 'select(.type=="assistant") | .message.content[]? | select(.type=="tool_use") | .name' "$F" | sort | uniq -c

# usage in [START, END): one record per message.id, main agent and subagents separately
jq -s --arg s "$START" --arg e "$END" '
  def ts: .timestamp | sub("\\.[0-9]+"; "") | fromdateiso8601;
  def known: (.input_tokens|type) == "number" and (.cache_creation_input_tokens|type) == "number"
             and (.cache_read_input_tokens|type) == "number"
             and (.input_tokens + .cache_creation_input_tokens + .cache_read_input_tokens) > 0;
  ($s|fromdateiso8601) as $S | ($e|fromdateiso8601) as $E
  | [ .[] | select(.type == "assistant" and ts >= $S and ts < $E) ]
  | group_by(.message.id // .uuid) | map(.[0])
  | group_by(.isSidechain == true)
  | map([ .[].message.usage | select(. != null and known) ] as $k
        | { agent: (if .[0].isSidechain == true then "subagent" else "main" end),
            responses: length,
            usage_unknown: (length - ($k | length)),
            input: ($k | map(.input_tokens) | add),
            cache_write: ($k | map(.cache_creation_input_tokens) | add),
            cache_read: ($k | map(.cache_read_input_tokens) | add),
            output: "unknown" })' "$F"
```

When `usage_unknown` is above 0, the sums cover only the known responses — say so in the report.

## Codex CLI

- **Path**: `${CODEX_HOME:-~/.codex}/sessions/YYYY/MM/DD/rollout-<timestamp>-<id>.jsonl` (newer builds may compress to `.jsonl.zst`; read with `zstd -dc`). A SQLite state index may sit next to it; the JSONL is the source of truth.
- **First line**: `session_meta` with `payload.cwd`, `payload.id`, `payload.cli_version`. One file is one session.
- **Usage**: `event_msg` with `payload.type == "token_count"`; `payload.info.total_token_usage` is a **cumulative snapshot** for the session, and its `input_tokens` **includes** `cached_input_tokens`. `info` is null on rate-limit-only events. The same snapshot can be emitted more than once.
  - Session total: the last snapshot of that file.
  - Period usage: last snapshot before the period end − last snapshot before the period start (counting rule 3). Don't add `last_token_usage` across events.
  - `reasoning_output_tokens`, when present, works the same way.
- **Cache hit** = `cached_input_tokens / input_tokens`.
- **Turns / model**: `task_started` / `task_complete` / `turn_aborted`; model in `turn_context`.
- **Tools**: `response_item` entries such as `function_call` and `web_search_call`.
- **Compaction**: `context_compacted` events.
- **Skills**: discovered from `.agents/skills` (cwd up to repo root) and user/admin/system locations; a skill was used when its `SKILL.md` was read or it was mentioned as `$skill-name`.

```bash
# usage in [START, END), per session, then summed
find "${CODEX_HOME:-$HOME/.codex}/sessions" -name 'rollout-*.jsonl*' -newermt "$START" | while read -r f; do
  case "$f" in *.zst) zstd -dc "$f" ;; *) cat "$f" ;; esac |
  jq -sc --arg s "$START" --arg e "$END" --arg f "$f" '
    def ts: .timestamp | sub("\\.[0-9]+"; "") | fromdateiso8601;
    ($s|fromdateiso8601) as $S | ($e|fromdateiso8601) as $E
    | (.[0] | ts) as $started
    | [ .[] | select(.type == "event_msg" and .payload.type == "token_count" and .payload.info != null)
            | {t: ts, u: (.payload.info.total_token_usage
                          | {input: .input_tokens, cached: .cached_input_tokens, output: .output_tokens})} ] as $snaps
    | ([ $snaps[] | select(.t < $S) ] | last) as $base
    | ([ $snaps[] | select(.t < $E) ] | last) as $end
    | if $end == null or $end.t < $S then empty
      elif $base == null and $started < $S then {file: $f, status: "unknown: no snapshot before period start"}
      else ($base.u // {input: 0, cached: 0, output: 0}) as $b
        | {file: $f, status: "ok"} + ($end.u | with_entries(.value -= $b[.key]))
      end'
done | jq -s '{sessions: length,
               unknown: (map(select(.status != "ok")) | length),
               input:  (map(select(.status == "ok") | .input)  | add),
               cached: (map(select(.status == "ok") | .cached) | add),
               output: (map(select(.status == "ok") | .output) | add)}'
```

Sessions listed as unknown are left out of the sums; report how many.

## Pi (pi-coding-agent)

- **Path**: `~/.pi/agent/sessions/--<cwd with "/" as "-">--/<timestamp>_<uuid>.jsonl`. Overridden by `PI_CODING_AGENT_DIR`, `PI_CODING_AGENT_SESSION_DIR`, `--session-dir`, or the `sessionDir` setting — check these first.
- **Records**: `type: "message"` with `message.role`; sessions are trees (branching via parent ids), so follow the active branch before counting turns. Model changes and compaction appear as their own entries; usage, when present, is on assistant messages — confirm on a sample whether it is per message or cumulative, then apply the counting rules.
- **Skills**: `~/.pi/agent/skills`, `~/.agents/skills`, project `.pi/skills`; invoked as `/skill:name`.

## OpenCode

- **Path**: SQLite at `~/.local/share/opencode/opencode.db` (older versions: JSON files under `~/.local/share/opencode/storage/`). Session diffs and tool output live in sibling folders.
- **Read**: `sqlite3 -readonly ~/.local/share/opencode/opencode.db '.tables'` then `.schema <table>` before querying; sessions, messages and parts are separate tables, and subagent sessions point to a parent session. Check whether token counts sit on messages or on parts before adding them, so nothing is counted twice.
- **Built-ins**: `opencode stats` for totals; `opencode export <session-id>` for one session as JSON.

## Chat surfaces (claude.ai, Claude app, ChatGPT)

No token, cache or tool-level logs are exposed to the model. Use the chat-history tools available in the session for the qualitative parts (friction, skills that were or weren't used, delegation) and mark every metric unknown.
