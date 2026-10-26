# Where agent session logs live

Checked against public docs and tools as of October 2026. Formats change between versions: always inspect a few records before writing a parser, and treat any field below as a hint, not a contract.

## Claude Code

- **Path**: `~/.claude/projects/<encoded-cwd>/<session-id>.jsonl`, where the working directory has `/` and `.` replaced by `-` (e.g. `/home/me/app` → `-home-me-app`). Subagent transcripts are records with `isSidechain: true` and, in newer versions, separate files under `<session-id>/subagents/`.
- **Retention**: deleted after `cleanupPeriodDays` (default 30) — older periods may simply be gone.
- **Records**: one JSON object per line with `type` (`user`, `assistant`, `system`, `attachment`, …), `timestamp`, `sessionId`, `cwd`, `gitBranch`, `version`.
- **Model**: `message.model` on assistant records. Sessions can mix models (hand-offs, `/model`).
- **Usage**: `message.usage` → `input_tokens` (uncached only), `cache_creation_input_tokens`, `cache_read_input_tokens`, `output_tokens`. Some records carry null or zero usage (relayed or handed-off turns): count those as unknown, not as zero.
- **Cache hit** = `cache_read / (input + cache_creation + cache_read)`.
- **Tools**: `tool_use` blocks in assistant `message.content` (`name`, `input`); results arrive in the next user record (`toolUseResult`). Skills show up as `Skill` tool calls; subagents as `Agent` (older: `Task`).
- **Compaction**: `system` records with `subtype: "compact_boundary"`, followed by a summary user message.

```bash
F=~/.claude/projects/<dir>/<session>.jsonl
jq -c '{type, subtype}' "$F" | sort | uniq -c                       # record mix
jq -r 'select(.type=="assistant") | .message.model' "$F" | sort | uniq -c
jq -r 'select(.type=="assistant") | .message.content[]? | select(.type=="tool_use") | .name' "$F" | sort | uniq -c
jq -s '[.[] | select(.type=="assistant") | .message.usage | select(. != null)]
       | {input: (map(.input_tokens // 0) | add), write: (map(.cache_creation_input_tokens // 0) | add),
          read: (map(.cache_read_input_tokens // 0) | add), output: (map(.output_tokens // 0) | add)}' "$F"
# how many assistant records have no usage at all (report as unknown, not zero)
jq -s '[.[] | select(.type=="assistant") | .message.usage
        | select(. == null or (.input_tokens // 0) + (.cache_read_input_tokens // 0) + (.output_tokens // 0) == 0)] | length' "$F"
```

## Codex CLI

- **Path**: `${CODEX_HOME:-~/.codex}/sessions/YYYY/MM/DD/rollout-<timestamp>-<id>.jsonl` (newer builds may compress to `.jsonl.zst`; read with `zstd -dc`). A SQLite state index may sit next to it; the JSONL is the source of truth.
- **First line**: `session_meta` with `payload.cwd`, `payload.id`, `payload.cli_version`.
- **Usage**: `event_msg` with `payload.type == "token_count"`; `payload.info.total_token_usage` is **cumulative** per session and `input_tokens` **includes** `cached_input_tokens`. Take the last value per session, or difference consecutive values for per-turn numbers. `info` is null on rate-limit-only events.
- **Cache hit** = `cached_input_tokens / input_tokens`.
- **Turns / model**: `task_started` / `task_complete` / `turn_aborted`; model in `turn_context`.
- **Tools**: `response_item` entries such as `function_call` and `web_search_call`.
- **Compaction**: `context_compacted` events.
- **Skills**: discovered from `.agents/skills` (cwd up to repo root) and user/admin/system locations; a skill was used when its `SKILL.md` was read or it was mentioned as `$skill-name`.

```bash
jq -c 'select(.type=="event_msg" and .payload.type=="token_count" and .payload.info != null)
       | .payload.info.total_token_usage' rollout-*.jsonl | tail -1
```

## Pi (pi-coding-agent)

- **Path**: `~/.pi/agent/sessions/--<cwd with "/" as "-">--/<timestamp>_<uuid>.jsonl`. Overridden by `PI_CODING_AGENT_DIR`, `PI_CODING_AGENT_SESSION_DIR`, `--session-dir`, or the `sessionDir` setting — check these first.
- **Records**: `type: "message"` with `message.role`; sessions are trees (branching via parent ids), so follow the active branch before counting turns. Model changes and compaction appear as their own entries; usage, when present, is on assistant messages — confirm field names on a sample.
- **Skills**: `~/.pi/agent/skills`, `~/.agents/skills`, project `.pi/skills`; invoked as `/skill:name`.

## OpenCode

- **Path**: SQLite at `~/.local/share/opencode/opencode.db` (older versions: JSON files under `~/.local/share/opencode/storage/`). Session diffs and tool output live in sibling folders.
- **Read**: `sqlite3 -readonly ~/.local/share/opencode/opencode.db '.tables'` then `.schema <table>` before querying; sessions, messages and parts are separate tables, and subagent sessions point to a parent session.
- **Built-ins**: `opencode stats` for totals; `opencode export <session-id>` for one session as JSON.

## Chat surfaces (claude.ai, Claude app, ChatGPT)

No token, cache or tool-level logs are exposed to the model. Use the chat-history tools available in the session for the qualitative parts (friction, skills that were or weren't used, delegation) and mark every metric unknown.
