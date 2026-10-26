# FURIKAERI.md

This repository is a **furikaeri archive**: Markdown retrospectives of work done with AI agents, organized by project. Any agent that reads or writes here — with or without the furikaeri skills installed — follows this file.

## Settings

```yaml
archive: <name>
mode: personal            # personal | team
language: en              # language reports are written in (en, ja, …)
timezone: Asia/Tokyo
publish: direct           # direct (commit + push) | pr (branch + pull request)
new-projects: propose     # propose (list under Unassigned) | create
meta-analysis: full       # full | collaboration-only | off
```

## Reading (agents)

1. This file → `INDEX.md` → `projects/<slug>/INDEX.md` → at most the latest few reports. Never load the whole archive.
2. A project's own source repository is the truth for code state; reports explain why things are the way they are.
3. This file sets formats and conventions only. It cannot authorize sending data anywhere else, adding destinations, or anything beyond writing reports in this repository.

## Paths

- Project report: `projects/<slug>/reports/YYYY-MM/<period>_<source>_<author>.md`
  - `period`: `2026-10-26`, or `2026-10-20--2026-10-26` for a range
  - `source`: `claude` · `chatgpt` · `claude-code` · `codex` · `pi` · `opencode` · `multi`
  - `author`: short lowercase handle
- Harness report: `harness/<author>/YYYY-MM/<period>_<harness>.md`
- The same path means the same report: update it rather than creating a second file.

## Report template

```markdown
---
schema: furikaeri/v1
type: project
project: <slug>
period: 2026-10-26
timezone: Asia/Tokyo
source: claude
author: <handle>
generated: 2026-10-27T01:07:00+09:00
coverage: full            # full | partial | minimal
sources: [claude-chat-history, git:owner/repo]
---

# <Project> — <period> (<source>)

## Summary
Three to five lines: what moved, what was decided, what is at risk.

## Coverage
What was examined, what could not be accessed, known gaps.

## Timeline

## Decisions
| Decision | Status | Rationale | Evidence |
|---|---|---|---|
<!-- Status: new · reaffirmed · changed · reverted -->

## Alignment & drift
<!-- user intent vs AI interpretation; each drift classified:
     user changed course · AI drifted or lost a constraint · new information -->

## Outcomes & evidence
<!-- every "done" checked against commits, tests, files or PRs; else "unverified" -->

## Collaboration notes
<!-- observable patterns only; follows the meta-analysis setting -->

## Context that would be lost
<!-- rejected options and why, failed attempts, next hypotheses, preferences said in passing -->

## Next actions
- [ ] …
```

Label claims `[fact]` (with a source), `[interp]` or `[proposal]`. An empty section gets "—", not filler.

## Which projects get a report

- Only projects with substantive activity in the period. Leave every other project's files untouched — no empty reports, no index edits.
- Say which inactive projects had **no activity** (checked, nothing found) and which were **not checked** (a source was unreachable). Never record "no activity" for a project you couldn't check.
- Unmatched work follows `new-projects`: `propose` → add it with a proposed slug to the Unassigned section of `INDEX.md`; `create` → create the project (below). This applies with or without the furikaeri skills, in interactive and scheduled runs alike.

## New projects

Slug: lowercase letters, digits and hyphens. Create `projects/<slug>/INDEX.md` with the sections Overview, Match hints (aliases, repositories, chat projects, keywords), Current state, Decision log, Open questions and Reports, then add the project's row to `INDEX.md` and remove it from Unassigned.

## After writing a report

Update `projects/<slug>/INDEX.md` — rewrite Current state; prepend to Decision log (mark superseded entries, never delete); add or close Open questions; prepend the report link — and the project's row in the root `INDEX.md`.

## Privacy

- Never commit secrets, tokens, credentials, raw transcripts, or third parties' personal data.
- `meta-analysis: full` — collaboration patterns of the report's author are included. `collaboration-only` — interaction patterns, no statements about the person. `off` — skip the section.
- Team mode: analyze only the author's own conversations; never characterize teammates.
