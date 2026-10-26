<p align="center">
  <img src="docs/images/hero-en.svg" alt="furikaeri — retrospectives for AI agents and human–AI projects" width="100%">
</p>

<p align="center">
  <b>English</b> · <a href="README.ja.md">日本語</a>
</p>

<p align="center">
  <img alt="License: MIT" src="https://img.shields.io/badge/license-MIT-F2A33A">
  <img alt="Agent Skills standard" src="https://img.shields.io/badge/Agent%20Skills-standard-7FB2E0">
  <img alt="Markdown only" src="https://img.shields.io/badge/runtime-Markdown%20only-7CC7A4">
  <img alt="Works with" src="https://img.shields.io/badge/works%20with-Claude%20%C2%B7%20ChatGPT%20%C2%B7%20Codex%20%C2%B7%20Pi%20%C2%B7%20OpenCode-B5835A">
</p>

**furikaeri** (振り返り, *fu-ri-ka-e-ri*) is Japanese for looking back on what you did, so the next thing goes better.

This repository is four small [Agent Skills](https://agentskills.io) that do exactly that for AI work. They review **what you and your AI decided and built**, keep those reviews in a **Git archive**, let your agents **recall** them while they work, and review **how your agent itself worked**. Plain Markdown: no scripts, no database, nothing to install in your projects.

<p align="center">
  <img src="docs/images/skills-en.svg" alt="The four skills: furikaeri-project (write), furikaeri-recall (recall), furikaeri-archive (store), furikaeri-harness (tune)" width="100%">
</p>

| Skill | Role | Looks at | Gives you |
|---|---|---|---|
| **`furikaeri-project`** | write | your conversations, commits, and the archive's past decisions | a project report: decisions, alignment, drift, collaboration patterns, and the context that would otherwise be lost |
| **`furikaeri-recall`** | recall | the archive, read-only, while you work | a short brief when work starts, a flag before a proposal that clashes with a recorded decision, answers to "why did we…?" |
| **`furikaeri-archive`** | store | the archive repository | creates it, adds projects, rebuilds the indexes, prints the pointer lines for your agents |
| **`furikaeri-harness`** | tune | session logs: tool calls, skills, models, tokens and cache hits, compaction, subagents, friction | findings as `ADD / UPDATE / REMOVE / NO_CHANGE` proposals, plus repeated workflows worth turning into new skills |

## How it fits together

<p align="center">
  <img src="docs/images/flow-en.svg" alt="Claude chats, ChatGPT chats, agent logs and git history flow into furikaeri-project, which files reports into a Git archive; furikaeri-recall reads it back for agents at work and for the next review" width="100%">
</p>

- **The archive is the memory.** Each project has an `INDEX.md` with its recorded state, a decision log, and *match hints* (aliases, repos, keywords). Reviews read it first, so they can tell a new decision from a reversed one — and `furikaeri-recall` brings it into everyday work.
- **The archive explains itself.** `FURIKAERI.md` at its root holds the settings, file paths, report template and privacy rules. An agent without these skills — say, a ChatGPT scheduled task — can still follow it. A team changes the format by editing one file.
- **Each service reviews its own chats.** Claude can't read your ChatGPT history and vice versa, so each one files its own report into the same project, side by side.
- **Nothing gets stuffed into context.** Reports pile up in Git; agents read only the indexes and the few reports a task needs.
- **It is a partial record, and it knows it.** Plenty is decided where no agent can see. See [The archive is never the whole story](#the-archive-is-never-the-whole-story).

## Install

### Option A — one command

```bash
npx skills add Aero123421/furikaeri-skills --skill '*' -g
```

The [`skills` CLI](https://github.com/vercel-labs/skills) asks which agents to install into (Claude Code, Codex, OpenCode, Cursor and more). Add `-a claude-code -a codex` to choose them up front.

### Option B — copy the folders

```bash
git clone https://github.com/Aero123421/furikaeri-skills.git
cd furikaeri-skills

# Claude Code (OpenCode reads this folder too)
mkdir -p ~/.claude/skills && cp -r skills/* ~/.claude/skills/

# Codex, Pi, OpenCode
mkdir -p ~/.agents/skills && cp -r skills/* ~/.agents/skills/
```

| Agent | Skills folder | How to call it |
|---|---|---|
| Claude Code | `~/.claude/skills/` or `.claude/skills/` | `/furikaeri-project` |
| Codex | `~/.agents/skills/` or `.agents/skills/` | `$furikaeri-project` |
| Pi | `~/.agents/skills/` or `~/.pi/agent/skills/` | `/skill:furikaeri-project` |
| OpenCode | `~/.agents/skills/`, `~/.claude/skills/` or `~/.config/opencode/skills/` | just ask; it loads skills by name |

### Option C — Claude app and ChatGPT

Zip each skill folder and upload it as a custom skill:

```bash
cd skills && for s in furikaeri-*; do zip -r "../$s.zip" "$s"; done
```

- **Claude (claude.ai, desktop, mobile):** upload the zips in your Skills settings.
- **ChatGPT:** upload them where your plan supports skills. If it doesn't, you can skip installing: the [scheduled prompt](prompts/scheduled-daily.md) plus your archive's `FURIKAERI.md` carry the whole method.

> [!TIP]
> These skills are meant to run when you ask. In Claude Code you can enforce that by adding `disable-model-invocation: true` to the front matter (a Claude Code extension, not part of the standard).

## Quick start

**1. Create your archive** — one private repository for all your retrospectives.

```text
Set up a furikaeri archive at github.com/<you>/furikaeri-archive (private).
```

`furikaeri-archive` creates `FURIKAERI.md` and `INDEX.md` and asks a couple of questions (report language, timezone, personal or team).

**2. Add a project** — the match hints are how later runs recognize it.

```text
Add project "acme-api" to my archive.
Repo: github.com/<you>/acme-api. Also called "acme" or "API v2".
```

It also prints two short pointer lines — one for the project's `AGENTS.md`, one for ChatGPT/Claude memory — so your agents check the archive before working. See [Let your agents remember](#let-your-agents-remember).

**3. Look back.**

```text
furikaeri for today
```

You get a report in the chat. When you confirm, it's saved as `projects/acme-api/reports/2026-10/2026-10-26_claude_<you>.md`, and the project's `INDEX.md` is updated.

**4. (Optional) Set your defaults once.** Edit the **Defaults** block at the top of `furikaeri-project`, `furikaeri-recall` and `furikaeri-harness` with your archive (and handle), so you don't have to repeat them.

**5. (Optional) Make it nightly.** See [Daily automation](#daily-automation).

## Things you can say

| You say | Skill | What happens |
|---|---|---|
| "furikaeri" · "look back on today" | project | today, every project with activity |
| "furikaeri acme-api last week" | project | one project, Monday–Sunday of last week |
| "look back on Oct 20–26 and open a PR" | project | a date range, published as a pull request |
| "review how my agent worked this week" | harness | tool, model, cache, compaction and subagent metrics |
| "anything worth turning into a skill this month?" | harness | repeated workflows, with evidence |
| *(you start working on acme-api)* | recall | a five-line brief of the recorded state, once per session |
| "why did we drop Redis?" | recall | the decision, its date and source — or "not in the archive" |
| "add project …" · "reindex the archive" | archive | archive maintenance |

**Periods:** today (default) · yesterday · last 7 days · last week (Mon–Sun) · last month · a date · a range like `2026-10-01..2026-10-26`. Times are in your timezone.

## Daily automation

<p align="center">
  <img src="docs/images/daily-en.svg" alt="Every night a scheduled task wakes up, reviews yesterday's chats, and files a report in Git" width="100%">
</p>

1. In **Claude** or **ChatGPT**, open **Scheduled** and create a new task. A time like **01:07** works well (just off the hour).
2. Paste the prompt from [`prompts/scheduled-daily.md`](prompts/scheduled-daily.md) and fill in your archive, handle and source (`claude` or `chatgpt`).
3. Check the first run: the report's **Coverage** section tells you what the task could and couldn't see.

Set it up in both services and each project gets one report per source every morning. For coding agents, run the same prompt from cron, e.g. `claude -p` or `codex exec`.

## Let your agents remember

A record nobody reads back changes nothing. `furikaeri-recall` is the read-only side: while you work, your agent looks at the project's `INDEX.md` at three moments only —

1. when work on a project starts or resumes (a brief of at most five lines, once per session),
2. before it proposes a design, tool or direction (it flags a clash with a recorded decision once, then follows your answer),
3. when you ask what was decided, or why.

Skills are picked by their description, which isn't always enough. Paste the pointer lines that `furikaeri-archive` prints when you add a project:

```markdown
## furikaeri
Before starting work or proposing a design here, check `projects/acme-api/INDEX.md` in github.com/you/furikaeri-archive (furikaeri-recall). It is a partial record: what I say now wins.
```

```text
My project decisions are partly recorded in github.com/you/furikaeri-archive. When I start work on a project listed there, or before you propose a design for it, check its INDEX.md first. If it differs from what I say, ask — what I say now wins.
```

The first goes in the project's `AGENTS.md` or `CLAUDE.md`, the second in ChatGPT or Claude memory. Both say *when* to look, not *what* is there, so they stay short. `furikaeri-harness` can tell you whether recall actually fired before design proposals, so you can tune the wording with numbers.

## The archive is never the whole story

The archive holds only what agents could see and chose to write down. Decisions made in meetings, in Slack, in chats without a furikaeri task, by agents without these skills, or since the last report aren't in it — and reports are AI interpretations of varying quality. Every skill, and the archive's own `FURIKAERI.md`, reads it that way:

- **Not recorded ≠ didn't happen.** Agents say "the archive has no record of X", never "X wasn't decided".
- **Recorded ≠ true now.** What you say now outranks the archive. A mismatch is treated as a stale record, and the agent asks.
- **Only decisions are decisions.** `[interp]` lines and `[proposal]`s are never read back as decisions; only decisions you made or accepted enter the Decision log.
- **Small, labeled, and quiet.** Recall shows a few lines marked "from the archive, recorded up to <date>", raises a clash once, and stays silent when nothing applies — so the record informs the work without steering it.

## What a report looks like

```markdown
# acme-api — 2026-10-26 (claude)

## Decisions
| Decision | Status | Rationale | Evidence |
|---|---|---|---|
| Keep REST, drop the GraphQL spike | changed | latency target met without it | chat 14:20, PR #42 |

## Alignment & drift
- [fact] Asked for "minimal error handling"; the AI added a retry framework.
- [interp] AI drift: the constraint from 10-20 ("no new abstractions") was lost.

## Context that would be lost
- Rejected: caching at the gateway — the auth headers vary per user.
```

Every claim is labeled `[fact]` (with a source), `[interp]`, or `[proposal]`. A full example from this repo's own design session is in [`examples/`](examples/projects/furikaeri-skills/reports/2026-10/2026-10-26_multi_aero.md).

## The archive

```
furikaeri-archive/
├── FURIKAERI.md                    settings, paths, report template, privacy
├── INDEX.md                        all projects + match hints
├── projects/
│   └── acme-api/
│       ├── INDEX.md                current state · decision log · open questions
│       └── reports/2026-10/
│           ├── 2026-10-26_claude_you.md
│           └── 2026-10-26_chatgpt_you.md
└── harness/you/2026-10/            optional harness reports
```

Settings live in `FURIKAERI.md`:

```yaml
mode: personal          # personal | team
language: en            # report language
timezone: Asia/Tokyo
publish: direct         # direct (push) | pr (pull request)
new-projects: propose   # propose | create
meta-analysis: full     # full | collaboration-only | off
```

## Principles

- **Evidence over recollection.** Metrics come from logs or say "unknown". Every "done" is checked against commits, tests or files.
- **Fact, interpretation and proposal are labeled separately.**
- **Fewer, sharper instructions.** Findings separate an agent's slip from a defect in its instructions, and prefer narrowing or removing a rule over adding one.
- **Propose, don't self-edit.** The skills never rewrite your skills, `AGENTS.md` or `CLAUDE.md`; they propose diffs.
- **Coverage is explicit.** Missing logs and unreachable chats are reported, never filled in.
- **The record is partial.** It informs; what you say now decides.
- **People are not graded.** Collaboration notes describe observable patterns, follow your privacy setting, and never characterize teammates.

## FAQ

<details>
<summary><b>Do I need Python, Node or a server?</b></summary>

No. The skills are Markdown. When they analyze logs, the agent uses tools it already has (`git`, `jq`, `sqlite3`…) through one-off commands.
</details>

<details>
<summary><b>Where does my data go?</b></summary>

Only to the archive repository you choose. Keep it private. Secrets, tokens and raw transcripts are never committed.
</details>

<details>
<summary><b>Can a team share one archive?</b></summary>

Yes. Set `mode: team` and `publish: pr`. Reports carry each author's handle, nobody edits another author's reports, and with `meta-analysis: collaboration-only` the notes describe interaction patterns, not people.
</details>

<details>
<summary><b>Will the archive overrule what I tell my agent?</b></summary>

No. What you say now wins. If it differs from a recorded decision, `furikaeri-recall` asks once ("still true, or changed elsewhere?") and then follows you. The next retrospective records the change.
</details>

<details>
<summary><b>Will it change my AGENTS.md or my skills?</b></summary>

No. `furikaeri-harness` proposes exact diffs and a way to verify them next time. You decide.
</details>

<details>
<summary><b>Why are older sessions missing from the harness review?</b></summary>

Some agents delete transcripts. Claude Code, for example, keeps them 30 days by default (`cleanupPeriodDays`). Run furikaeri regularly, or raise the setting.
</details>

## Repository layout

```
skills/
├── furikaeri-harness/   SKILL.md · references/log-sources.md
├── furikaeri-project/   SKILL.md
├── furikaeri-recall/    SKILL.md
└── furikaeri-archive/   SKILL.md · assets/ (FURIKAERI.md, INDEX.md, PROJECT_INDEX.md)
prompts/                 scheduled-daily.md · scheduled-daily.ja.md
examples/                a sample project index and report
docs/images/             the illustrations in this README
```

## Contributing

Issues and pull requests are welcome — especially log formats for more agents ([`log-sources.md`](skills/furikaeri-harness/references/log-sources.md)), translations of the report template, and reports of what scheduled tasks can and can't see.

## License

[MIT](LICENSE)

<p align="center">
  <img src="docs/images/owl.svg" alt="the furikaeri owl" width="120"><br>
  <sub>Owls can turn their heads about 270°. Very good at looking back.</sub>
</p>
