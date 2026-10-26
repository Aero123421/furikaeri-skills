---
name: furikaeri-archive
description: Create or maintain a furikaeri archive — a Git repository of Markdown retrospectives organized by project, for one person or a team. Use to set up a new archive, add a project, rebuild the indexes, or check the archive's health. Reports themselves are written by furikaeri-project.
license: MIT
---

# furikaeri-archive

The archive is plain Markdown in Git. Its `FURIKAERI.md` tells every agent — Claude, ChatGPT, Codex, Pi, OpenCode — how to read and write it, so the archive works even where the furikaeri skills aren't installed.

```
<archive>/
├── FURIKAERI.md                 conventions, settings, report template
├── INDEX.md                     projects + match hints
├── projects/<slug>/
│   ├── INDEX.md                 current state, decision log, open questions, reports
│   └── reports/YYYY-MM/<period>_<source>_<author>.md
└── harness/<author>/YYYY-MM/    optional, from furikaeri-harness
```

## init

1. **Target**: the repository or local path the user gives. If it doesn't exist, create it only after the user confirms, private by default (e.g. `gh repo create <name> --private`), or ask the user to create it and share the URL.
2. Copy `assets/FURIKAERI.md` and `assets/INDEX.md` to the root. Fill the Settings block; ask only what you can't infer, with defaults: `mode: personal`, `language`: the user's, `timezone`: the user's, `publish`: `direct` for personal or `pr` for team, `new-projects: propose`, `meta-analysis: full` for personal or `collaboration-only` for team.
3. Commit `furikaeri: init archive` and push.
4. Offer next steps: add the first projects, and set up a scheduled task from the furikaeri `prompts/` folder.

## add-project

1. Choose a slug (lowercase letters, digits, hyphens). Make sure no existing project already covers it under another name or alias.
2. Create `projects/<slug>/INDEX.md` from `assets/PROJECT_INDEX.md`. Fill Overview and Match hints — aliases, repository URLs, Claude/ChatGPT project names, distinctive keywords — from the user or the conversation that prompted it. furikaeri-project assigns conversations using these hints, so make them concrete.
3. Add the project's row to the root `INDEX.md`, then commit and publish per the archive's `publish` setting.

## reindex

Rebuild the generated parts from the files on disk: the root project table (last report date, report count) and each project's Reports list, newest first. Never rewrite Current state or the Decision log — furikaeri-project curates those. Report broken links and files that don't follow the path conventions.

When a project `INDEX.md` grows past roughly 150 lines, move Decision log entries older than the current year to `projects/<slug>/decisions/YYYY.md` and link them, so the entry point stays small.

## health (on request)

Look for: projects with no report in a long while, Current state that contradicts the latest report, projects with overlapping aliases, conversations stuck under Unassigned. Propose fixes; apply only after the user confirms.

## Rules

- Markdown and Git only: no databases, generated JSON, embeddings or CI required.
- Never delete reports; newer reports supersede older ones.
- Team mode: never edit another author's reports; index changes go through the configured publish mode.
- Never commit secrets, tokens or raw transcripts.
