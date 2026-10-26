# furikaeri-skills

> Example project index, as `furikaeri-project` keeps it in an archive.

## Overview

Markdown-only Agent Skills for retrospectives of AI agents and human–AI projects, with a Git archive as shared memory.

## Match hints

- Aliases: furikaeri, 振り返りスキル, furikaeri skills
- Repositories: github.com/Aero123421/furikaeri-skills
- Chat projects: furikaeri
- Keywords: retrospective, harness review, FURIKAERI.md, archive

## Current state

- Recorded up to: 2026-10-26 (multi, coverage: partial)
- Goal: daily retrospectives from Claude and ChatGPT land in one Git archive, and every agent reads past decisions before writing new ones.
- Direction: three skills (harness, project, archive); Markdown only; conventions live in the archive's `FURIKAERI.md`.
- Constraints: nothing installed in work projects; no scripts, databases or CI; never self-edit skills or AGENTS.md.
- Status: v0.1 published; scheduled runs not yet tested.

## Decision log

- 2026-10-26 — Archive carries its own conventions in `FURIKAERI.md` — [report](reports/2026-10/2026-10-26_multi_aero.md)
- 2026-10-26 — Repo-first discovery: projects and context come from the archive — [report](reports/2026-10/2026-10-26_multi_aero.md)
- 2026-10-26 — Each service reviews its own chats into one shared archive — [report](reports/2026-10/2026-10-26_multi_aero.md)
- 2026-10-26 — ~~Project-side `.furikaeri/config.yaml`~~ reverted: Markdown only — [report](reports/2026-10/2026-10-26_multi_aero.md)

## Open questions

- [ ] How much chat history can a scheduled run see in Claude and in ChatGPT?
- [ ] Can a ChatGPT scheduled task push to GitHub without stopping for approval?

## Reports

- [2026-10-26 · multi · aero](reports/2026-10/2026-10-26_multi_aero.md)
