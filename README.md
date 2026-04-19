# Docket

Hand off tasks to AI coding agents as plain markdown files — one per task, status and questions tracked inline, git-friendly.

## Quick start

In a fresh repo, tell your AI coding agent (Claude Code, Cursor, Codex, Gemini CLI, etc.):

> Read https://raw.githubusercontent.com/poteb/Docket/main/SETUP.md and scaffold Docket in this repo.

The agent fetches the setup instructions and sets everything up in one pass. Then queue your first task.

## What it is

Docket is a convention, not a tool. You write tasks into a `Tasks/` folder. Your AI coding agent (Claude Code, Cursor, Codex, Gemini CLI, etc.) picks them up, updates status in the file, logs questions, and moves completed tasks to `Tasks/done/`. No daemon, no UI, no dependencies — just markdown and git.

## Why

Chat with an agent is ephemeral. File-based tasks persist across sessions, survive context compaction, give you a queue you can edit with your normal editor, and produce a clean git history of what was asked and how it went.

## Layout

Once adopted, your project looks like this:

```
your-project/
├── Tasks/
│   ├── 001-scaffold.md         # todo | in progress | blocked
│   ├── 002-add-thumbnails.md
│   └── done/
│       └── 000-initial.md      # completed tasks archived here
├── Done.md                     # flat append-only log
└── AGENTS.md                   # or CLAUDE.md — contains the Docket rules
```

## The rules

1. **One file per task**, in `Tasks/`, named `NNN-short-slug.md` (3-digit zero-padded prefix).
2. **You create task files** in the main conversation with Status `todo` and a description.
3. **Execution is delegated to a sub-agent** so the main conversation stays free for queueing more tasks. When you say "Proceed", a sub-agent picks up the lowest-numbered `todo` task. "Proceed with task 5" targets a specific one.
4. **The sub-agent does the bookkeeping**: Status → `in progress` → `done`, appends progress to `## Notes`, moves the file to `Tasks/done/`, and logs a one-line summary to `Done.md`.
5. **Questions**: the sub-agent sets Status to `blocked`, writes the question in `## Questions`, and returns. The main conversation flags it in chat.
6. **Auto-proceed**: when a task finishes cleanly, the next `todo` task starts automatically in a fresh sub-agent. Stops on `blocked`, errors, empty queue, or when you say "stop".

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

For Claude Code users, Docket ships a project-local skill at [`.claude/skills/add-docket/`](.claude/skills/add-docket/) that automates task creation — it scans `Tasks/` and `Tasks/done/` to pick the next number, derives a kebab-case slug, runs a clarity check on the description (and asks back if anything is vague), and writes the file. See [`SETUP.md`](SETUP.md) step 3 for installation.

## License

MIT — see [`LICENSE`](LICENSE).
