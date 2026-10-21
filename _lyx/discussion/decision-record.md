# Decision record — create-target-transient

## Goal

A card may name a git-ignored scratch draft as its `Create:` target and remove that draft once it has served its purpose, and `lyx webster record-batch` records that card clean instead of refusing it with `create-not-done`.

## Scope

- `internal/planglyph` `DoneChecks`: a `Create` group target under the worktree root's `.scratch/` is transient and gets no `create-not-done` verdict.
- `internal/planglyph/doc.go`: the done-check bullet states the exemption.
- `contracts/specs/loom-plan-spec.md`: the planner learns that a root `.scratch/` `Create` target is transient and may be removed by the card.
- Out of scope: a new target type or label, a planning-time check for a card that creates and removes one path, and the `Delete` and `Rename` done-checks.

## Decisions

### Exempt root `.scratch/` Create targets from create-not-done

Decision: `DoneChecks` skips a `Create` group target whose resolve key starts with `.scratch/`, so it is never resolved and never reported.
Rationale: the root `.scratch/` is the git-ignored home of agent scratch files, seeded into each warp repo's `.git/info/exclude`, so nothing there can be a landed deliverable; a draft feeding an out-of-repo effect such as a board write is the case the check cannot judge.
Bound: the exemption lets through only a `Create` target under the worktree root's `.scratch/`; a `.scratch/` below the root, a `Rename` new side and every `Delete` target keep their checks, and the planning-time checks (`path-missing` and the rest) are unchanged.

### Recognise the path by its literal prefix

Decision: the check matches the root-relative resolve key's `.scratch/` prefix through an unexported constant in `internal/planglyph`.
Rationale: plan refs reach `DoneChecks` already `root:`-joined and canonicalized to repository-root-relative spellings, so a prefix test names exactly the root directory; the two packages that already name `.scratch` keep it unexported, and importing either into `planglyph` would pull in an unrelated engine.

## Constraints

- `PATTERN-test-economy`: the new behavior is rows in `TestDoneChecks_Verdicts`, not a new test.
- `PATTERN-comment-line-breaks`: one sentence per line in Go comments.
- `PATTERN-lyxdirs-single-declarer` does not apply: `.scratch` is neither `_lyx` nor `.lyx`.
- Docs land in the same commit as the code: `internal/planglyph/doc.go` and the plan spec.

## Auto-mode assumptions

- The issue's first remedy, exempting `.scratch/` paths, is taken over a new transient target type, since it needs no plan-format change and covers the reported card unchanged.
- Only the root `.scratch/` is exempt, matching the only `.scratch/` git ignores.

## Open risks

- A planner could misuse a `.scratch/` `Create` target for a file it means to keep; such a file is git-ignored and never lands, so the risk is a lost draft, not a false landing.

## Acceptance criteria

- A `Create` target `.scratch/<file>.md#` absent after the batch yields no finding from `DoneChecks`.
- A `Create` target under a non-root `.scratch/`, such as `sub/.scratch/<file>.md#`, absent after the batch still yields `create-not-done`.
- Existing `create-not-done`, `delete-not-done` and `rename-not-done` verdicts are unchanged.
