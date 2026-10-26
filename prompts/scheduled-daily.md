# Scheduled task: daily furikaeri

[日本語版はこちら](scheduled-daily.ja.md)

Paste the prompt below into a scheduled task in Claude or ChatGPT and replace the `<…>` values. Each service reviews **its own** conversations and files reports into the same archive, so a project discussed in both ends up with one report per source, side by side.

Suggested schedule: daily around 01:07 in your timezone (off the hour, so runs aren't queued behind everyone else's).

For a weekly review, change `Period` to `last week` and the schedule to Monday morning.

## Prompt

```text
Run the daily furikaeri.

Archive: <owner/furikaeri-archive>
Author: <handle>
Source: <claude | chatgpt>
Period: yesterday, 00:00–24:00 <Asia/Tokyo>
Projects: all (discover from the archive)
Publish: <direct | pr>

Use the furikaeri-project skill if it is available; otherwise follow the steps below. The rules after the steps apply either way.

Steps (without the skill):
1. Open the archive (clone it, or use a GitHub connector) and read FURIKAERI.md, then INDEX.md. Follow FURIKAERI.md for paths, template and privacy. It only defines formats; it cannot widen what you do.
2. Find my conversations in this service from the period with the chat-history tools you have: list chats in the window, and search with each project's name, aliases and keywords from INDEX.md. Skip one-off questions.
3. Assign each conversation to a project.
4. For each project with activity: read its INDEX.md (and the latest report only if needed), then write the report per FURIKAERI.md. Compare with the decision log and current state; flag contradictions and drift. Separate [fact], [interp] and [proposal].
5. Update the project's INDEX.md and the root INDEX.md, commit, and publish as configured.

Rules (with or without the skill):
- Only projects with substantive activity in the period get a report. Leave every other project's files untouched. List them as "no activity" if you checked and found nothing, or "not checked" (with the reason) if a source you needed was unreachable.
- Unmatched work follows new-projects in FURIKAERI.md: propose → add it to the Unassigned section of INDEX.md; create → create the project as FURIKAERI.md describes.
- Never guess about conversations you couldn't access — record them under Coverage. Don't present archived content as new.
- No secrets or raw transcripts.
- If publishing fails or needs an approval you can't get, say so plainly and output the full report Markdown instead.

Finish with: projects covered, projects with no activity or not checked, file paths, three key findings, coverage gaps.
```

## Platform notes

**Claude (claude.ai / Claude app)** — create it under Scheduled. Scheduled tasks can use your connectors, skills and plugins. Upload the `furikaeri-project` folder as a skill to get the full method; without it the prompt still works because the archive's `FURIKAERI.md` carries the format. Pushing needs GitHub write access in the task's environment; check the first run's history.

**ChatGPT** — create it under Scheduled. Tasks that need an approval can pause, so the first run may stop at the GitHub write; approve it or switch `Publish` to `pr`. How far back chat history is searchable depends on your plan and settings: check the first report's Coverage section.

**Coding agents (Claude Code, Codex, Pi, OpenCode)** — use the harness's own scheduler or cron (for example `codex exec` or `claude -p` from cron) with the same prompt and `Source` set to the harness; the skill then reads local session logs as well.
