## Goal

```markdown
Imported from [issue #185](https://github.com/Knatte18/loomyard/issues/185).

Seen: on a loom run in Webster, lym loom decision add --amends-plan sent the run back to Plan-Burler-Review. Webster's state.json, which recorded four committed batches, was lost. Plan-Burler-Fix then failed with create-already-exists on cards 1-4, because their Create entries had already landed on the task branch.

Workaround: the parent ruled by decision that the landed Creates become Edit entries, and validate-plan then passed.

Expected: an amendment during Webster keeps the record of committed batches, and the plan rework sees which cards already landed, so it neither re-creates their files nor needs a hand ruling.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-23): Untraceable incident driven in a test

Decision: The incident in issue #185 cannot be traced to a run. The reported path is driven in a test, and the test lands when it shows no loss of the Webster run record.

Rationale: No run in this hub matches the report; a test of the path is the only evidence available.

### Added after Discussion (parent, 2026-11-23): No test seam: the reported path loses no run record

Decision: TestAmendsPlanDuringWebster_KeepsDoneBatches passes on the merge base because no step of the reported path (decision add --amends-plan, goto Plan-Burler-Review, Plan-Burler-Fix) moves _lym/webster/state.json; only Plan-Write's and Parts-Plan-Write's rotation, webster run --fresh and webster reset --to start do. The test lands as a guard, not as a fix.

Rationale: The red-first gate expects a test that fails on base; here there is no bug on the path to show.

### Added after Discussion (parent, 2026-11-23): Accept round 1: F1 fixed, F2 fixed differently

Decision: Accept the code review at round 1. F1 (webster-state-filename-literal) is fixed: the test reads the record through websterengine.LoadState before and after the goto. F2 is fixed as the review's own fallback allows.

Rationale: Mechanical test fixes, visible in internal/loomcli/amendduringwebster_test.go; another round would only confirm them.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
