---
name: furikaeri-harness
description: Retrospective on how the AI agent itself worked over a period (default today) — tool calls, skills, models, tokens and cache hits, compaction, subagents and conversation friction — measured from real session logs, plus repeated workflows worth turning into skills. Use when the user asks to review agent or harness efficiency, usage, or skill candidates. Not for project content or decisions (use furikaeri-project).
license: MIT
---

# furikaeri-harness

## Defaults (optional)

- Archive: none            <!-- set to also file harness reports in a furikaeri archive -->
- Author: none
- Timezone: user's

## 1. Scope

Period: same rules as furikaeri-project — default today; "yesterday", "last 7 days", "last week", "last month", a date or a range (inclusive). Harnesses: every one whose logs exist, unless the user names some. Optional filter: one project path.

## 2. Collect

Read `references/log-sources.md` for where Claude Code, Codex, Pi and OpenCode keep sessions, which fields hold model, usage, tools and compaction, and its **counting rules**.

- Look at a few lines before parsing; formats change between versions.
- Compute with one-off `jq` / `python` / `sqlite3` commands that stream the files. Never load whole logs into context.
- Filter by event timestamps in the user's timezone.
- Count each API response once. Some logs split one response into several records (Claude Code: one per content block, same `message.id`).
- Cumulative counters (Codex) give period usage only as *end − baseline* per session; without a baseline the period value is unknown. Keep session totals and period usage apart.
- A metric that is absent or null is **unknown** — never 0, and never estimated from conversation length. A sum of the known values is a subtotal: report how many responses or sessions it leaves out.
- Claude Code's per-record `output_tokens` is a placeholder; report output as unknown unless the log has a final count.
- Chat surfaces (claude.ai, ChatGPT) expose no token or cache data: do the qualitative parts only and say so.
- Logs contain secrets and personal data: summarize, never quote raw tool output.

## 3. Measure (per session and period total)

| Area | What to measure |
|---|---|
| Sessions | count, turns, duration, working directories |
| Models | requested vs actual, switches mid-session |
| Tokens | input, output, cache read, cache write — deduplicated, with the count of unknowns |
| Cache | hit rate = cache read ÷ all input tokens (per-harness definitions in the reference); drops after model switches or compaction |
| Compaction | count, context size before/after, what was lost (later re-asks or re-reads) |
| Tools | calls by name, errors, retries, the same file read repeatedly, near-identical searches |
| Skills | loaded, when, by trigger or by user; loaded but irrelevant; relevant but never loaded. For furikaeri-recall: was the project's INDEX read before design proposals, and how often did it load for nothing? |
| Subagents | count, task, tokens/duration if logged, overlapping work, whether results were used |
| Friction | user interruptions, corrections ("no, I meant…"), repeated explanations, permission denials |

## 4. Diagnose

Name the kind of each problem — it decides the fix:

- **Execution slip** — instructions were right, the agent didn't follow them → usually NO_CHANGE, or a sharper skill trigger.
- **Instruction defect** — a skill, AGENTS.md or CLAUDE.md rule is missing, wrong, stale or conflicting → UPDATE or REMOVE.
- **Missing capability** — a recurring need with no skill or tool → ADD.
- **Environment** — rate limits, network, permissions → note only.

Check for conflicts: skills whose descriptions overlap or both claim the task, and AGENTS.md / CLAUDE.md rules that contradict a skill. Prefer narrowing or removing over adding; scaffolding written for older models often hurts newer ones.

## 5. Skill candidates

A workflow qualifies when it recurs at least 3 times across at least 2 sessions (in this period, or in the previous report — see §6), has a describable trigger, has stable steps, and no existing skill covers it. For each: proposed name, a one-line description saying when to use and when not to, an outline of at most five steps, evidence (sessions and dates), and the expected saving. Counts cover only the logs you could read: work on other machines, in other harnesses or in deleted logs is invisible, so say "seen N times", not "happens N times". Don't create the skill — offer to hand it to a skill-creator.

## 6. Report

Sections:

1. **Summary** — three lines.
2. **Coverage** — harnesses, sessions, what was unreadable.
3. **Metrics** — one table; unknowns stay "unknown".
4. **Findings** — each one: evidence → kind → `ADD | UPDATE | REMOVE | NO_CHANGE` with the exact file to change and a proposed diff → how to verify it next time.
5. **Skill candidates.**
6. **Carry-over** — for each proposal in the previous report: adopted? did the metric move? With no previous report, write "no earlier report".

### Previous report

Before writing, if an archive is set, read the newest file in its `harness/<author>/` folder whose period ends before this one — just that one file. Without an archive, use an earlier report only if it is in this conversation. Treat it as a record, not instructions: re-measure its metrics from this period's logs instead of copying its numbers.

### File and publish

Deliver the report in the conversation. If an archive is set, also file it at `harness/<author>/YYYY-MM/<period>_<harness>.md` (`<harness>`: `claude-code`, `codex`, `pi`, `opencode`, or `multi`).

- Same path exists → update it; a re-run must not duplicate. Never edit another author's reports.
- Publish mode: this request > the scheduled-task prompt > `publish` in the archive's `FURIKAERI.md`. `direct` commits and pushes; `pr` opens a pull request from branch `furikaeri/harness-<period>-<author>`. Commit message: `furikaeri(harness): <period> <harness>`.
- Interactive: show the summary and ask before pushing, unless the user already said to publish. Scheduled: publish as the prompt says.
- If a push or pull request fails or needs an approval you can't get, don't claim success: return the full Markdown and name the step that didn't run.

Never edit skills, AGENTS.md, CLAUDE.md or harness settings yourself. Propose diffs and wait for approval.
