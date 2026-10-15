---
verdict: APPROVED
---

## Alignment at pr-gate: plan-test-file-path

The diff of pull request 650 at `e7d235d1e` against `main` matches the board entry's goal and the decision record.

- Goal: the plan stencil drops the plain test-file exception under `**Edit:**`, and the new `edit-test-file` check in `internal/planparser/edittestfile.go` refuses a `_test.go` file self glyph under `**Edit:**` with D5's way forward.
- D1 and D2: the stencil states that a new test's `plan:` handle names the package directory or `<dir>_test`, never a file, and adds the `**Uses:**` rule for reused test setup, helpers and fakes.
- D3 and D4: the check is blocking, skips `language: none` and paths the go tool ignores, takes the done set at format validation and none at rework, is never appended at dispatch, has its own `refGate` and ledger entry in `shape.go`, and reads the path through `Glyph.UnitPath`; `loom-plan-spec.md` and `internal/planparser/doc.go` drop the disk-existence-only limit on `Glyph.UnitPath`.
- D5: the spec's row 43 and the finding's detail carry the way forward.
- D6: no Webster stencil or pack code changes.
- The post-Discussion addition: `caller-uncovered` and `redundant-file-target` offer only the member glyph for a `_test.go` file, and the Plan-Review rubric sentence and its claim are gone, the claim now a `mustNot`.
- Out-of-scope items stay out: no other target group is gated, no non-test `.go` path is gated, the pack audit is not committed, and stencil wording claims are updated rather than loosened, leaving that to `lighter-test-rules`.

The open risks carried into the decision record:

- Plan-Review `stencil-caller-rule-offers-test-file`: the stencil now says a caller in a `_test.go` file is listed by the member glyph only (`loom-template-plan.md`, after the caller rule).
- Webster-Review `rubric-resign-arrow-bullet-merged`: commit `35b1124c2` restores the re-sign arrow bullet's own line; the diff against `main` only removes the test-file sentence above it.
- The decision record's own open risk, a fork placing a new test in a new file, is a recorded and accepted bound, not a deviation.

The board entry's Idea says the `**Create:**` declaration "carries the file"; D1 records that the handle names the package and the fork picks the file, and the diff follows D1.

## Overlaps

`lyx loom peers` lists no parent commits and two peers.

- `engine-registry` (row 14) and `creel-core` (row 17): each shares only `cmd/lyx/testdata/test-budget.yaml` with this run, a shared bookkeeping file; this run changes the `internal/planparser` row, which neither peer touches by path.
  Landing order: this run, at row 22 (`own_row`), is furthest along and lands first; each peer takes the change at its Merge-In.

## Related board entries

- `lighter-test-rules` depends on this task and will remove the wording claims this run updates; the decision record defers that to it, so there is no conflict.
- `webster-deviations-list` rewords the fork's deviation rule later; D6 leaves it unconstrained and this diff does not touch it.

No board entry is made obsolete by this run.
