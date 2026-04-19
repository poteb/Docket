# Docket

Hand off tasks to AI coding agents as plain markdown files — one per task, status and questions tracked inline, git-friendly.

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

## Adopting Docket in your project

1. Paste the snippet below into your project's agent instructions file — `CLAUDE.md` for Claude Code, `AGENTS.md` for Codex and most generic agents, `.cursorrules` for Cursor, `GEMINI.md` for Gemini CLI.
2. Create empty `Tasks/` and `Tasks/done/` folders (drop a `.gitkeep` in `done/` so git tracks it).
3. Create an empty `Done.md`.
4. Write your first task at `Tasks/001-whatever.md` using the template above — or use the `add-docket` skill (see below) if you're on Claude Code.
5. Say "Proceed".

### Snippet to paste

`````markdown
## Task workflow (Docket)

Tasks live as one file per task under `Tasks/`, named `NNN-short-slug.md` (3-digit zero-padded prefix, e.g. `001-scaffold.md`).

- **The user creates** task files in the main conversation. Initial Status is `todo` with a description of the work.
- **Execution is always delegated to a sub-agent**, so the main conversation stays free for the user to queue more tasks. When the user says "Proceed", dispatch a sub-agent to take the lowest-numbered `todo` task. "Proceed with task 5" dispatches for that specific task.
  *(Claude Code: use the `Agent` tool with `subagent_type: general-purpose` and `run_in_background: true`.)*
- **The sub-agent does the bookkeeping**: Status → `in progress`, append progress/decisions/findings to `## Notes` as it works, Status → `done` when finished, move the file from `Tasks/` to `Tasks/done/`, and append a one-line summary to `Done.md` (newest at bottom, format: `NNN — short description of what was done`).
- **If the sub-agent needs input**, it sets Status to `blocked`, writes the question in `## Questions`, and returns. The main conversation flags it in chat so the user can answer. Auto-proceed pauses until the user responds.
- **Auto-proceed**: when a sub-agent finishes a task cleanly (status `done`), immediately dispatch the next-lowest-numbered `todo` task in a fresh sub-agent, with no prompt from the user. Surface a one-line "task N done; starting task N+1" update in chat. Stop auto-proceeding when: (a) no `todo` tasks remain, (b) a task ends `blocked`, (c) the sub-agent errors or fails verification, or (d) the user says "stop" or "pause".
- **Brief the sub-agent well**: sub-agents start with no conversation context. The dispatch prompt must tell the sub-agent to read its task file at `Tasks/NNN-*.md`, follow these workflow rules, and include any decisions from prior tasks it needs. Point it at relevant files by path.

### Status vocabulary
`todo` | `in progress` | `blocked` | `done`

### Task file template

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
`````

## Optional: the `add-docket` skill

For Claude Code users, Docket ships a project-local skill at [`.claude/skills/add-docket/`](.claude/skills/add-docket/) that automates task creation — it scans `Tasks/` and `Tasks/done/` to pick the next number, derives a kebab-case slug, runs a clarity check on the description (and asks back if anything is vague), and writes the file.

Install it in your project without cloning this repo:

```bash
curl -sL --create-dirs -o .claude/skills/add-docket/SKILL.md \
  https://raw.githubusercontent.com/poteb/Docket/main/.claude/skills/add-docket/SKILL.md
```

Or copy the folder directly from a local clone of this repo.

## License

MIT — see [`LICENSE`](LICENSE).
