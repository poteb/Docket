# Docket SETUP

Scaffolding instructions for installing Docket in a project. An AI coding agent can execute these steps end-to-end from this document alone. If anything already exists, skip that step — don't duplicate or overwrite.

## Steps

1. **Create the directory structure** at the repo root:
   - Folder `Tasks/` (empty)
   - File `Tasks/done/.gitkeep` (empty — creates the folder and lets git track it)
   - File `Done.md` (empty — the flat append-only log)

2. **Add the Docket workflow rules to your agent's instructions file.** Append the snippet in the next section to whichever file your agent reads (create it if missing, don't duplicate if it's already there):
   - `CLAUDE.md` — Claude Code
   - `AGENTS.md` — Codex and most generic agents
   - `.cursorrules` — Cursor
   - `GEMINI.md` — Gemini CLI

3. **(Optional — Claude Code only) Install the `add-docket` skill**, which automates task creation:

   ```bash
   curl -sL --create-dirs -o .claude/skills/add-docket/SKILL.md \
     https://raw.githubusercontent.com/poteb/Docket/main/.claude/skills/add-docket/SKILL.md
   ```

4. **Report back** what was created (files/folders added, which agent-instructions file was updated, whether the skill was installed). Then wait for the user's first task.

## Snippet to paste

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
