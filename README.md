# Docket

Hand off tasks to AI coding agents as plain markdown files — one per task, status and questions tracked inline, git-friendly.

## Quick start

In a fresh repo, tell your AI coding agent (Claude Code, Cursor, Codex, Gemini CLI, etc.):

> Read https://raw.githubusercontent.com/poteb/Docket/main/SETUP.md and scaffold Docket in this repo.

The agent fetches the setup instructions and sets everything up in one pass. Then queue your first task.

## What it is

Docket is a convention, not a tool. You write tasks into a `Tasks/` folder. Your AI coding agent (Claude Code, Cursor, Codex, Gemini CLI, etc.) picks them up, updates status in the file, and moves completed tasks to `Tasks/done/`. No daemon, no UI, no dependencies — just markdown and git.

Filenames are UTC-timestamp-prefixed (`YYYYMMDD-HHMMSS-slug.md`), so multiple contributors can create tasks in parallel without coordination or merge conflicts.

Each task also carries a **Severity** (`trivial` | `minor` | `major`) that decides how much process it gets: a colour tweak is just done and merged, a small feature is grilled and test-driven, a big feature gets a spec, a plan and the full review treatment.

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
2. **You create task files** in the main conversation with Status `todo`, a Severity and a description.
3. **Execution is always delegated to a sub-agent — never run in the main conversation.** The main conversation stays free for queueing more tasks. When you say "Proceed", a sub-agent picks up the **earliest** `todo` task (smallest filename = oldest-created). "Proceed with `<slug>`" targets a specific one by slug.
4. **The sub-agent does the bookkeeping**: Status → `in progress` → `done`, appends progress to `## Notes`, and moves the file to `Tasks/done/`. `Tasks/done/` listed chronologically IS the log — no separate file.
5. **Questions**: the sub-agent sets Status to `blocked`, writes the question in `## Questions`, and returns. The main conversation flags it in chat.
6. **Auto-proceed**: when a task finishes cleanly, the next `todo` task starts automatically in a fresh sub-agent. A task blocked on a `### Grilling` question round doesn't stop the queue — it's flagged in chat and the next `todo` starts. Stops on any other `blocked`, errors, empty queue, or when you say `Halt` (let in-flight sub-agent(s) finish first, then no more dispatches) or `stop`/`pause` (cancel in-flight sub-agent(s) immediately; cancelled tasks stay at Status `in progress` for you to revert to `todo`, edit, or delete).

7. **Severity** decides how the sub-agent executes the task (see [Severity tiers](#severity-tiers)). It's set at creation and can be changed any time; a sub-agent that finds the work bigger than rated raises it and says why, but only you lower it.
8. **Grilling** resolves a `minor`/`major` task's open questions before work starts — see [Grilling](#grilling).

## Severity tiers

| Severity | Covers | How it's executed |
|---|---|---|
| `trivial` | A colour, a text/copy change, a config value, a one-line fix whose correctness is visible at a glance | Just do it, verify (screenshot if visible), merge. No grilling, no review chain. |
| `minor` | A small feature, or a bug fix with real logic, bounded to one area | Grilling → test-driven implementation → self-review → independent agent review → an external-model review only when the change is risky (money, auth, security, privacy, data loss/migrations, webhooks, or the agent review found serious issues) → merge. No spec, no plan. |
| `major` | A big or fuzzy feature, anything cross-cutting, or a design-level change to money / auth / security / data | Spec + plan, doubt-driven development (adversarial review of each non-trivial decision), security hardening (threat model + checklist), and the full review chain including the external model. |

Severity decides *which steps run*. Your project's own rules (review channels, merge gates, branch/worktree conventions) still apply on top. The step-by-step recipes the sub-agent follows live in the skill's [`SEVERITY.md`](.claude/skills/add-docket/SEVERITY.md).

## Grilling

A rough `minor`/`major` idea can be filed as-is with Grilling `pending`; its open questions get resolved by one of three routes, all ending with `## Grilling` set to `resolved` and the decisions listed under it:

1. **Now** — grill the idea in the main conversation when you file it.
2. **Prep pass** — say "grill the dockets" (e.g. before an overnight run) and the main conversation walks every task still pending grilling with you, dispatching nothing. Then "Proceed" runs the queue unattended.
3. **By the sub-agent** — a dispatched task that still has Grilling `pending` reads the code, writes its questions under `### Grilling` in `## Questions`, and blocks. Answer them and the task goes back to `todo`.

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

## Severity
minor  <!-- trivial | minor | major — set at creation; may be updated later -->

## Task
<what needs to be done>

## Grilling
pending  <!-- n/a (trivial) | pending | resolved — resolved decisions listed below -->

## Notes
<agent appends progress, decisions, findings as work proceeds>

## Questions
<optional; only when status is blocked>
````

## Adopting Docket

See [`SETUP.md`](SETUP.md) for step-by-step scaffolding instructions — or use the Quick start above to have an agent do it for you.

## Optional: the `add-docket` skill

For Claude Code users, Docket ships a project-local skill at [`.claude/skills/add-docket/`](.claude/skills/add-docket/) that automates task creation and runs the queue — it generates the UTC timestamp prefix, derives a kebab-case slug, rates the Severity, runs a clarity check (or offers grilling for `minor`/`major` ideas), writes the file, handles prep passes, and briefs each sub-agent with its tier recipe from [`SEVERITY.md`](.claude/skills/add-docket/SEVERITY.md). See [`SETUP.md`](SETUP.md) step 3 for installation.

The tier recipes call other Claude Code skills by name: `grilling`, `tdd`, `code-review`, `feature` and `domain-modeling` (from [mattpocock-skills](https://github.com/mattpocock/skills) and the author's own set), and [`doubt-driven-development`](https://github.com/addyosmani/agent-skills/tree/main/skills/doubt-driven-development) / [`security-and-hardening`](https://github.com/addyosmani/agent-skills/tree/main/skills/security-and-hardening) from addyosmani/agent-skills. Swap in your own equivalents if you use different ones.

## Migrating from sequential numbering

Earlier versions of Docket used `NNN-slug.md` filenames and a separate `Done.md` log. If you're upgrading:

- Leave historical files in `Tasks/done/` as-is — they're archival. No retroactive renaming needed.
- If `Done.md` exists, keep it as a historical record or delete it — the new convention doesn't write to it.
- New tasks use the `YYYYMMDD-HHMMSS-slug.md` format going forward. The skill, when updated, generates the new format automatically.
- Verbal references change: "Proceed with task 5" → "Proceed with `<slug>`". Bare "Proceed" still works exactly as before.
- Tasks created before Severity existed have no `## Severity` section; a `todo` one gets rated (and the section added) when it's next dispatched or prepped.

## License

MIT — see [`LICENSE`](LICENSE).
