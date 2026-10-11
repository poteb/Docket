# Docket Severity tiers — execution recipes

You are a sub-agent executing one Docket task. Read `## Severity` in your task file and run **only that tier's recipe** below. The project's `CLAUDE.md` / memory still governs worktrees, sandbox rules, review *channels* (which model reviews, Loom vs. API), merge gates and round budgets; where it names a concrete tool for a step (e.g. "Claude review = Fable 5.1 agent", "ChatGPT = Loom"), use that tool for the step here.

**Re-rating:** if the work turns out bigger than its tier, raise `## Severity`, write why in `## Notes`, and switch to the higher recipe. If the higher tier needs grilling that hasn't happened, run the grilling question round (below) instead. Never lower Severity yourself.

**Grilling question round** (any tier that needs `## Grilling: resolved` and finds `pending`): read the relevant code first, then write your open questions as a design tree under `### Grilling` in `## Questions` — each question with your recommended answer — set Status `blocked`, and return. Ask only what the code and project docs can't answer.

## trivial — just do it

1. Make the change on a branch (project worktree rules).
2. Verify: compile/build the affected area. For a visible change, take a screenshot of the result (the project's preview recipe; both sizes if the project asks for them) and check it shows the intended change.
3. Open the PR with the screenshot, run the project's merge gate, and merge. No grilling, no review chain, no Loom.
4. Done when: merged, screenshot in the PR, task file in `Tasks/done/`.

## minor — small feature, no spec

Precondition: `## Grilling` is `resolved`; otherwise run the grilling question round.

1. Implement with the `tdd` skill: red → green → refactor per behaviour, the tests asserting the decisions in `## Grilling`. No spec document, no plan document.
2. Self-review the full diff against the project's coding standards (the `code-review` skill); fix what it finds.
3. Independent review: a fresh-context Claude agent (the project's named reviewer model, if any), read-only, severity-tagged P1/P2/P3, given the diff + task file — not your reasoning. Fix valid findings.
4. ChatGPT / Loom review **only when risk-triggered**: the change touches money or payments, auth or permissions, security, privacy / personal data, data loss or migrations, webhooks or idempotency — or step 3 found any P1/P2. Record in `## Notes` whether it triggered and why. Rounds per the project's budget.
5. Merge when every review that ran is clean: run the project's merge gate, then merge.
6. Done when: merged, task file in `Tasks/done/`, `## Notes` lists reviews run + findings fixed/rejected.

## major — big or fuzzy feature, the full works

Precondition: `## Grilling` is `resolved` and holds an approved design (from `feature` steps 1–2 and 5 in the main conversation); otherwise run the grilling question round.

1. Run the `feature` skill from step 3: write the spec (folding in `## Grilling`), spec review, plan, implement (TDD), PR. Use the project's ChatGPT channel (e.g. Loom) wherever `feature` says ChatGPT, and the project's merge policy instead of its final "notify for human review" step when the project grants merge authority.
2. **Doubt-driven**: apply `doubt-driven-development` to every non-trivial decision (its definition) while planning and implementing — fresh-context adversarial reviewer, ARTIFACT + CONTRACT only, classify findings, max 3 cycles. You are non-interactive: skip its cross-model offer and say so; the ChatGPT/Loom review is the cross-model pass.
3. **Security hardening**: apply `security-and-hardening` — threat model (trust boundaries, assets, STRIDE, abuse cases as first tests) before implementing, and walk its checklist before sign-off. Record the threat model in the spec or PR body.
4. **Full review**: self-review (`code-review`) → independent Claude agent review → ChatGPT / Loom, always. Rounds per the project's budget.
5. Merge per the project's merge policy (merge gate first).
6. Done when: merged (or handed to the user, if the project withholds merge authority), spec + threat model linked from `## Notes`, task file in `Tasks/done/`.
