---
schema: furikaeri/v1
type: project
project: furikaeri-skills
period: 2026-10-26
timezone: Asia/Tokyo
source: multi
author: aero
generated: 2026-10-26T22:30:00+09:00
coverage: partial
sources: [chatgpt-conversation (exported and pasted), claude-conversation, git:Aero123421/furikaeri-skills]
---

# furikaeri-skills — 2026-10-26 (multi)

> **Example report.** Written from the real design session that produced this repository, so you can see the format filled with real content.

## Summary

- [fact] Designed three Markdown-only skills (harness, project, archive) and published v0.1 as this repository.
- [fact] The design changed direction twice, both times on the user's correction: no project-side `.furikaeri/` config, and the archive becomes the *source* of project context, not only the destination.
- [interp] Biggest open risk: how much chat history a scheduled run can actually see. Unknown until the first real nightly run.

## Coverage

- ChatGPT: one conversation, full text, exported and pasted into Claude (not read through ChatGPT's own history tools).
- Claude: the hand-off session.
- Not examined: any earlier chats about the idea.

## Timeline

1. ChatGPT surveys harness and context-engineering discussion as of 2026-10-26.
2. The idea is split into three skills; accepted.
3. Correction: no `.furikaeri/` folder; Markdown only. New: Claude/ChatGPT scheduled tasks push a daily report to Git.
4. Correction: read the archive's projects first and pull their context ("repo-first discovery").
5. Hand-off to Claude: fact-check, finalize the design, write v0.1, publish.

## Decisions

| Decision | Status | Rationale | Evidence |
|---|---|---|---|
| Skills are Markdown only: no scripts, no config files | new | portability across harnesses; user requirement | ChatGPT turn 3 |
| `.furikaeri/config.yaml` in each project | reverted | user does not want project-side files | ChatGPT turns 2–3 |
| Each service reviews its own chats; one shared archive | new | no service can read another's history; keeps provenance | ChatGPT turn 3 |
| Project `INDEX.md` = match hints + current state + decision log | new | lets any run find projects and compare with past decisions | ChatGPT turn 4, Claude session |
| Archive carries its own conventions in `FURIKAERI.md` | new | agents without the skills (e.g. a ChatGPT task) can follow the format; teams customize in one place | Claude session |
| Third skill named `furikaeri-archive` (was `furikaeri-init`) | new | it also adds projects and rebuilds indexes | Claude session |

## Alignment & drift

- [fact] The second proposal introduced `.furikaeri/config.yaml`; the user rejected it.
- [interp] Classified as **AI drift**: the opening request asked for *simple* skills, and the proposal added project-side infrastructure.
- [fact] Repo-first discovery arrived as a new requirement from the user. Classified as **user changed course**, not drift.
- [fact] Several research claims from the ChatGPT session could not be verified on 2026-10-26 (two named papers and the CompactionBench figures). They were kept out of the design and the README.

## Outcomes & evidence

- [fact] v0.1 published to `Aero123421/furikaeri-skills` (initial commit).
- [fact] The Claude Code log snippet was run on a real transcript, counting each response once: cache reads were about 96% of input tokens over 11 responses; 4 responses had no usage and are reported as unknown, not zero. Output tokens: unknown (the per-record value is a placeholder).
- [fact] Three issues were filed against v0.1 the same day (#1–#3): double-counted Claude Code usage, Codex period totals, and project selection that differed with and without the skill. All three were fixed.
- Unverified: scheduled runs in Claude and ChatGPT; Pi and OpenCode log fields on real sessions.

## Collaboration notes

- [fact] Requirements surfaced mostly through corrections: two of four user turns in the ChatGPT session redirected the design.
- [interp] The assistants leaned toward adding infrastructure (config files, per-harness variants); the user consistently pulled toward fewer moving parts.
- [proposal] Open future design prompts with the standing constraint: "Markdown only, nothing installed in the project."

## Context that would be lost

- Rejected: a `/furikaeri` router skill. Each skill is invoked directly, so an alias adds a fourth file for no gain.
- Rejected for now: parsers, databases, embeddings. Revisit only if an archive reaches tens of thousands of reports.
- Hypothesis: ChatGPT scheduled tasks may pause at the GitHub write for approval; `publish: pr` may be smoother there.
- Preference: built for personal use first, but team use must work (one archive, many authors).

## Next actions

- [ ] Create a private archive and run the nightly task in both Claude and ChatGPT for a week.
- [ ] Check what chat-history search returns inside scheduled runs, and record it in Coverage.
- [ ] Verify Pi and OpenCode log fields on real sessions; update `log-sources.md`.
