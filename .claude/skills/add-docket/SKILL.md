---
name: add-docket
description: Use when the user asks to create, add, queue, file, or docket a new task — phrases like "make a task for X", "add a task to do Y", "add to the docket", "docket this", or "new task" — drives the Docket task queue with "Proceed", "Proceed with <slug>", "Halt", "stop", or "pause", or asks to grill / prep the dockets (e.g. before an overnight run). Applies in any project that follows the Docket convention (a `Tasks/` folder with one markdown file per task).
---

# add-docket

The Docket task workflow: create task files in `Tasks/` (UTC timestamp prefix, a **Severity** rating, and a clarity check or grilling), and run the queue by dispatching sub-agents that execute each task by its Severity tier. Creating and grilling tasks happens in the main conversation; executing them never does — see [Workflow rules](#workflow-rules).

## When to use

- User asks to create / add / queue / file a new task.
- User says "make a task to X", "add a task for Y", "turn this into a task".
- User wants to turn a verbal request into a persistent task file.
- User says "Proceed" / "Proceed with `<slug>`" (dispatch a task) or "Halt" / "stop" / "pause" (stop the queue) — see [Workflow rules](#workflow-rules).
- User says "grill the dockets" / "prep the dockets" / "prepare for an overnight run" — see [Prep pass](#prep-pass).

Do NOT use for: editing existing tasks or listing tasks. Those don't need this skill.

## Creating a task

1. **Ensure `Tasks/` exists.** From the repo root, check for `Tasks/`. If missing, create `Tasks/` and `Tasks/done/` (drop a `.gitkeep` in `done/` so git tracks it).

2. **Generate a UTC timestamp** with PowerShell (the Bash tool's clock runs ~32h behind on Paw's PC):

   ```powershell
   [DateTime]::UtcNow.ToString('yyyyMMdd-HHmmss')
   ```

   Example output: `20260419-145321`. This is the filename prefix — no scan of existing files needed, timestamps are collision-free by construction.

3. **Rate the Severity** (`trivial` | `minor` | `major`) by the definitions in [Severity](#severity). Pick it yourself; the user can override it in the report line.

4. **Clarity check.** Evaluate the description against these criteria:
   - **Goal**: Is it clear what "done" looks like?
   - **Scope**: Is the boundary of the work defined (what's in, what's out)?
   - **Inputs**: Does the task reference files, data, or decisions the user hasn't provided?
   - **Design choices**: Are there forks-in-the-road you'd have to guess at?

   - **`trivial`**: if any of these is ambiguous, **ask the user before writing the file**. Use AskUserQuestion when there are 2+ discrete choices; use a direct chat question for a single open-ended gap. Do not invent answers — a vague task file is worse than no task file, because future-you will act on it.
   - **`minor` / `major`**: a rough idea is fine to file — open questions are what [Grilling](#grilling) resolves. Write down what the user said, then ask one question: grill now, or file it with Grilling `pending` (grilled later in a prep pass or by the sub-agent's question round). Ask before writing only when even the goal is unknown.

5. **Derive a slug.** Short kebab-case, 2–5 words, lowercase, hyphen-separated. Strip articles ("the", "a", "an") and filler ("please", "can you"). Example: "Add thumbnails to the sidebar" → `add-thumbnails-sidebar`.

6. **Write the file** at `Tasks/<timestamp>-<slug>.md`:

````markdown
# <Title Case of slug>

## Status
todo  <!-- todo | in progress | blocked | done -->

## Severity
minor  <!-- trivial | minor | major — set at creation; may be updated later (see Severity) -->

## Task
<Cleaned-up description: 1–3 sentences or a short bullet list. Keep the user's intent; trim filler.>

## Grilling
pending  <!-- n/a (trivial) | pending | resolved — resolved decisions listed below -->

## Notes

## Questions
````

   If the exact filename already exists (same second + same slug — extremely rare), bump the timestamp by 1 second and retry.

7. **Report** the created file path and Severity back to the user in one line, e.g. `Created Tasks/20260419-145321-add-thumbnails-sidebar.md (minor, grilling pending).`

## Clarity check — examples

**Clear enough, write it:**
> "Add a dark-mode toggle to the header that persists to localStorage."

Goal, scope, and behavior are all specified.

**Vague, ask first (`trivial`) or grill (`minor`/`major`):**
> "Make the scripts faster."

Which scripts? By how much? What's acceptable to change?

> "Add thumbnails."

Where? Sourced from what? One size or responsive?

> "Clean up the UI."

Unbounded. Push the user for a specific, acceptance-testable slice.

## Severity

Decides how a task is executed. The tier recipes the sub-agent follows live in [SEVERITY.md](SEVERITY.md); the dispatcher points the sub-agent at its tier there.

| Severity | What it covers | Execution in one line |
|---|---|---|
| `trivial` | Small tweaks: a colour, a text/copy change, a config value, a one-line fix with obvious correctness | Just do it, verify (screenshot if visible), auto-merge, done. No grilling, no review chain. |
| `minor` | A small feature, or a bug fix with real logic, bounded to one area | Grilling → TDD → self-review → independent Claude agent review → ChatGPT/Loom only when risk-triggered → auto-merge. No spec, no plan. |
| `major` | A big or fuzzy feature, anything cross-cutting, or a design-level change to money / auth / security / data | `feature` skill (spec + plan) with grilling, `doubt-driven-development`, `security-and-hardening`, full review incl. ChatGPT/Loom; merge per the project's merge policy. |

- **Updating**: the user may change Severity at any time. A sub-agent that finds the task bigger than rated raises it, writes why in `## Notes`, and continues under the new tier — or goes `blocked` when the new tier needs grilling that hasn't happened. Lowering Severity is the user's call only.
- **Project rules still apply on top**: the project's `CLAUDE.md` / memory decides the review *channels* (which model reviews, Loom vs. API), merge gates, worktree rules and round budgets. Severity decides *which steps run*.

## Grilling

Resolves a `minor` / `major` task's open questions before execution. `trivial` tasks are `n/a`. Three routes, all ending with `## Grilling` set to `resolved` and the decisions listed under it:

1. **Now, at creation**: in the main conversation. `minor` → the `grilling` skill. `major` → `feature` skill steps 1–2 and 5 (brainstorm an approved design, then grill it with `domain-modeling`). Fold the outcome into `## Grilling`.
2. **Later, in a [prep pass](#prep-pass)**: same as route 1, for tasks filed with Grilling `pending`.
3. **By the sub-agent**: a dispatched task with Grilling `pending` gets a question round — the sub-agent reads the code, writes its questions as a design tree in `## Questions` under a `### Grilling` heading, sets Status `blocked`, and returns. When the user answers, the main conversation folds the answers into `## Grilling`, sets it `resolved` and Status back to `todo`.

### Prep pass

"Grill the dockets" / "prep the dockets" (e.g. before an overnight run): in the main conversation, walk every `todo` task with Grilling `pending` and every task `blocked` on a `### Grilling` question round, oldest first, and grill each with the user via route 1. Dispatch nothing. Finish with a one-line-per-task readiness list (`ready` / `skipped by user`), so "Proceed" can then run the queue unattended.

## Filename rules

- Timestamp is always UTC, format `YYYYMMDD-HHMMSS` — ASCII-sortable and collision-free in practice.
- Always run `[DateTime]::UtcNow.ToString('yyyyMMdd-HHmmss')` (PowerShell) live. Don't hard-code, don't reuse a previous timestamp, don't estimate.
- The slug is the *verbal handle* — short, memorable, lowercase kebab-case. Long slugs (>5 words) should be trimmed.
- Never retroactively rename a task file; the creation timestamp is immutable.

## Common mistakes

| Mistake | Fix |
|---|---|
| Writing a `trivial` file before clarifying vague asks | Ask first. Better to stall 30 seconds than create a misleading task. |
| Interrogating the user before filing a rough `minor`/`major` idea | File it with Grilling `pending`; grill now only if the user chooses it. |
| Rating a feature `trivial` to skip the review chain | `trivial` is for changes whose correctness is visible at a glance; any new logic is at least `minor`. |
| Hard-coding or estimating the timestamp | Always run `[DateTime]::UtcNow.ToString('yyyyMMdd-HHmmss')` live. |
| Using local time | Always UTC (`UtcNow`). Mixed timezones break sort order across contributors. |
| Long slugs (>5 words) | Trim. The timestamp is the identity; the slug is just a hint. |
| Status `in progress` on a new task | Always start as `todo`. |
| Echoing the user's raw words into `## Task` verbatim | Clean up: trim filler, tighten to 1–3 sentences. Keep the intent. |
| Creating the task file when you really should have just answered the question | If the user is asking something, not queuing work, don't write a file. |

## Workflow rules

These rules govern the whole Docket lifecycle, not just creation:

- **One file per task**, in `Tasks/`, named `YYYYMMDD-HHMMSS-slug.md` (UTC timestamp + kebab-case slug). Timestamps keep filenames unique across contributors without coordination.
- **Task files are created in the main conversation** with Status `todo`, a Severity and a description (that's the creation workflow above).
- **Execution is always delegated to a sub-agent — never run in the main conversation.** The main conversation stays free for queueing and grilling. When the user says "Proceed", a sub-agent picks up the **earliest** `todo` task (smallest filename = oldest-created). "Proceed with `<slug>`" targets a specific one by slug.
- **The sub-agent does the bookkeeping**: Status → `in progress` → `done`, appends progress to `## Notes`, and moves the file to `Tasks/done/`. `Tasks/done/` listed chronologically IS the log — no separate file.
- **Questions**: the sub-agent sets Status to `blocked`, writes the question in `## Questions`, and returns. The main conversation flags it in chat.
- **Auto-proceed**: when a task finishes cleanly, the next `todo` task starts automatically in a fresh sub-agent. A task that ends `blocked` on a `### Grilling` question round does not stop the queue — flag it in chat and dispatch the next `todo`. Stops on any other `blocked`, errors, empty queue, or when the user says `Halt` (let in-flight sub-agent(s) finish first, then no more dispatches) or `stop`/`pause` (cancel in-flight sub-agent(s) immediately; cancelled tasks stay at Status `in progress` for the user to revert to `todo`, edit, or delete).
- **No auto-resume**: only `todo` tasks are ever dispatched — never resume an `in progress` task (e.g. one left behind by a cancelled sub-agent) unless the user explicitly asks. With no sub-agent in flight, `Halt` and `stop` are equivalent — both just prevent the next dispatch.
- **Status vocabulary**: `todo` | `in progress` | `blocked` | `done`.
- **Legacy repos**: earlier Docket versions used `NNN-slug.md` filenames and a separate `Done.md` log, and tasks without a `## Severity` section. Leave historical files as-is (no retroactive renaming), and don't write to `Done.md` — `Tasks/done/` is the log. A `todo` task without Severity gets rated (and the section added) when it is next dispatched or prepped. New tasks always use the timestamp format. Targeting is by slug ("Proceed with `<slug>`"), not by number.

### Dispatching (Claude Code)

- Dispatch with the `Agent` tool, `subagent_type: general-purpose`, `run_in_background: true`. Cancel in-flight sub-agents with `TaskStop`.
- **Brief the sub-agent well**: it starts with no conversation context. The dispatch prompt must tell it to read its task file by path, follow the bookkeeping rules above, and execute the task's Severity tier from `~/.claude/skills/add-docket/SEVERITY.md` (re-reading `## Severity` first, since the user may have changed it). Include any decisions from prior tasks it needs. Point it at relevant files by path.
