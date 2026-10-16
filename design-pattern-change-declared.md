# A task that changes PATTERN declares it in its record and plan

`pattern-change-declared` · Note · bug, stencil, loom, webster, burler

## Seen

Found by the stencil intent audit (`docs/research/stencil-intent-audit-2026-10-15/findings.md`, F4); line numbers as of `main` on 2026-10-16.

Every role that gets the pattern directive (plan writer, Plan-Review, the Webster master and so its forks, the burler reviewer, the Bouncer judge) gets `PATTERN.md` as it stands at the worktree root when its prompt is rendered.
The reviewer and judge directives make every cited violation BLOCKING and never downgradable (`contracts/stencils/pattern/pattern-directive-review-fix.md:18`, `pattern-directive-judge.md:17`).
This repository's `CLAUDE.md` puts a new invariant into `PATTERN` in the same commit as its code, so a task that changes a PATTERN entry is judged against the old text: a reviewer flags the very change the task is meant to make as a violation.
The webster master reads `PATTERN.md` and the `pattern/` files once and its forks inherit them; `contracts/stencils/webster/webster-template-master.md:31` claims those reads "stay current for the whole run because the run refuses a drifted plan", which covers the plan but not a card that edits `PATTERN.md`.
The Discussion already names each entry a design changes (`pattern-directive-designer.md:18`), but nothing carries that to the later roles as a sanctioned change.

## Decision

Operator decision (2026-10-16): a task that changes PATTERN says so explicitly in its decision record, as a clear "this changes PATTERN" statement naming each entry and its new rule, so every reviewer knows.
The edit itself still happens in Webster, not before.
The plan carries the same statement, so its reviewers raise no conflict either.

Operator view, not a decision: Plan-Review reads the decision record anyway.
(Agent's check: it does, as its answer key; Webster-Review measures against the plan only, so the plan's copy is what reaches it.)

## Idea

Agent's suggestions:

- A fixed section in the decision record, defined in `loom-discussion-spec.md` and checked by `validate-discussion`, listing each PATTERN entry the task adds, changes or removes with its new text.
- A matching plan-level section in `00-overview.md`, required by the plan gate whenever the record's section is non-empty.
- The reviewer, judge, implementer and orchestrator pattern directives say that an entry the task lists as changed is judged against the listed new text, never the old.
- The webster master's claim at `:31` is narrowed to what the drift check covers.
- The same treatment may suit `CLAUDE.md`, which cards also edit; the operator has not decided that.
