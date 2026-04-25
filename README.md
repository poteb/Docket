# Docket

Hand off tasks to AI coding agents as plain markdown files — one per task, status and questions tracked inline, git-friendly.

## Quick start

In a fresh repo, tell your AI coding agent (Claude Code, Cursor, Codex, Gemini CLI, etc.):

> Read https://raw.githubusercontent.com/poteb/Docket/main/SETUP.md and scaffold Docket in this repo.

The agent fetches the setup instructions and sets everything up in one pass. Then queue your first task.

## What it is

Docket is a convention, not a tool. You write tasks into a `Tasks/` folder. Your AI coding agent (Claude Code, Cursor, Codex, Gemini CLI, etc.) picks them up, updates status in the file, and moves completed tasks to `Tasks/done/`. No daemon, no UI, no dependencies — just markdown and git.

Filenames are UTC-timestamp-prefixed (`YYYYMMDD-HHMMSS-slug.md`), so multiple contributors can create tasks in parallel without coordination or merge conflicts.

## Why

Chat with an agent is ephemeral. File-based tasks persist across sessions, survive context compaction, give you a queue you can edit with your normal editor, and produce a clean git history of what was asked and how it went.

## Layout

Once adopted, your project looks like this:

```
your-project/
├── Tasks/
│   ├── 20260419-145321-add-thumbnails.md         # todo | in progress | blocked
│   ├── 20260419-150044-scaffold-dark-mode.md
│   └── done/
│       └── 20260418-093000-initial.md            # completed tasks archived here
└── AGENTS.md                                     # or CLAUDE.md — contains the Docket rules
```

Chronological order of `Tasks/done/` (by filename) is the completion log — no separate log file needed.

## The rules

1. **One file per task**, in `Tasks/`, named `YYYYMMDD-HHMMSS-slug.md` (UTC timestamp + kebab-case slug). Timestamps keep filenames unique across contributors without coordination.
2. **You create task files** in the main conversation with Status `todo` and a description.
3. **Execution is always delegated to a sub-agent — never run in the main conversation.** The main conversation stays free for queueing more tasks. When you say "Proceed", a sub-agent picks up the **earliest** `todo` task (smallest filename = oldest-created). "Proceed with `<slug>`" targets a specific one by slug.
4. **The sub-agent does the bookkeeping**: Status → `in progress` → `done`, appends progress to `## Notes`, and moves the file to `Tasks/done/`. `Tasks/done/` listed chronologically IS the log — no separate file.
5. **Questions**: the sub-agent sets Status to `blocked`, writes the question in `## Questions`, and returns. The main conversation flags it in chat.
6. **Auto-proceed**: when a task finishes cleanly, the next `todo` task starts automatically in a fresh sub-agent. Stops on `blocked`, errors, empty queue, or when you say `Halt` (let in-flight sub-agent(s) finish first, then no more dispatches) or `stop`/`pause` (cancel in-flight sub-agent(s) immediately; cancelled tasks stay at Status `in progress` for you to revert to `todo`, edit, or delete).

## Status vocabulary

- `todo` — not started
- `in progress` — agent is working on it
- `blocked` — agent has a question; see `## Questions` in the file
- `done` — finished; file moves to `Tasks/done/`

## Task file template

````markdown
# <title>

## Status
todo  <!-- todo | in progress | blocked | done -->

## Task
<what needs to be done>

## Notes
<agent appends progress, decisions, findings as work proceeds>

## Questions
<optional; only when status is blocked>
````

## Adopting Docket

See [`SETUP.md`](SETUP.md) for step-by-step scaffolding instructions — or use the Quick start above to have an agent do it for you.

## Optional: the `add-docket` skill

For Claude Code users, Docket ships a project-local skill at [`.claude/skills/add-docket/`](.claude/skills/add-docket/) that automates task creation — it generates the UTC timestamp prefix, derives a kebab-case slug, runs a clarity check on the description (and asks back if anything is vague), and writes the file. See [`SETUP.md`](SETUP.md) step 3 for installation.

## Migrating from sequential numbering

Earlier versions of Docket used `NNN-slug.md` filenames and a separate `Done.md` log. If you're upgrading:

- Leave historical files in `Tasks/done/` as-is — they're archival. No retroactive renaming needed.
- If `Done.md` exists, keep it as a historical record or delete it — the new convention doesn't write to it.
- New tasks use the `YYYYMMDD-HHMMSS-slug.md` format going forward. The skill, when updated, generates the new format automatically.
- Verbal references change: "Proceed with task 5" → "Proceed with `<slug>`". Bare "Proceed" still works exactly as before.

## License

MIT — see [`LICENSE`](LICENSE).
