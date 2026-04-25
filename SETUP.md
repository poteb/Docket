# Docket SETUP

Scaffolding instructions for installing Docket in a project. An AI coding agent can execute these steps end-to-end from this document alone. If anything already exists, skip that step — don't duplicate or overwrite.

## Steps

1. **Create the directory structure** at the repo root:
   - Folder `Tasks/` (empty)
   - File `Tasks/done/.gitkeep` (empty — creates the folder and lets git track it)

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

4. **Report back** what was created (folders added, which agent-instructions file was updated, whether the skill was installed). Then wait for the user's first task.

## Snippet to paste

`````markdown
## Task workflow (Docket)

Tasks live as one file per task under `Tasks/`, named `YYYYMMDD-HHMMSS-slug.md` (UTC timestamp + kebab-case slug, e.g. `20260419-145321-scaffold.md`). The timestamp keeps filenames unique across contributors without coordination. `Tasks/done/` (the archive) IS the log — listed chronologically by filename.

- **The user creates** task files in the main conversation. Initial Status is `todo` with a description of the work.
- **Do NOT execute tasks in the main conversation.** Execution is always delegated to a sub-agent, so the main conversation stays free for the user to queue more tasks. When the user says "Proceed", do not open the task file and start working — dispatch a sub-agent instead. It takes the **earliest** `todo` task (smallest filename = oldest-created). "Proceed with `<slug>`" dispatches for the task whose slug matches.
- **Claude Code dispatch**: use the `Agent` tool with `subagent_type: general-purpose` and `run_in_background: true`. Other agents: use your platform's equivalent sub-agent / background-task mechanism.
- **The sub-agent does the bookkeeping**: Status → `in progress`, append progress/decisions/findings to `## Notes` as it works, Status → `done` when finished, and move the file from `Tasks/` to `Tasks/done/`.
- **If the sub-agent needs input**, it sets Status to `blocked`, writes the question in `## Questions`, and returns. The main conversation flags it in chat so the user can answer. Auto-proceed pauses until the user responds.
- **Auto-proceed**: when a sub-agent finishes a task cleanly (status `done`), immediately dispatch the next-earliest `todo` task in a fresh sub-agent, with no prompt from the user. Surface a one-line "`<slug>` done; starting `<next-slug>`" update in chat. Stop auto-proceeding when: (a) no `todo` tasks remain, (b) a task ends `blocked`, (c) the sub-agent errors or fails verification, or (d) the user says `Halt`, `stop`, or `pause`.
- **Halt vs stop**: `Halt` is graceful — do not dispatch any further `todo` tasks, but let in-flight sub-agent(s) run to completion. `stop` and `pause` are immediate — cancel in-flight sub-agent(s) using the platform's cancel mechanism (e.g. Claude Code's `TaskStop`). A cancelled task stays in `Tasks/` with Status `in progress` and whatever partial `## Notes` were written; the user decides whether to revert it to `todo`, edit it, or delete it. Sub-agents do not auto-resume `in progress` tasks. With no sub-agent in flight, `Halt` and `stop` are equivalent — both just prevent the next dispatch.
- **Brief the sub-agent well**: sub-agents start with no conversation context. The dispatch prompt must tell the sub-agent to read its task file by path, follow these workflow rules, and include any decisions from prior tasks it needs. Point it at relevant files by path.

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
