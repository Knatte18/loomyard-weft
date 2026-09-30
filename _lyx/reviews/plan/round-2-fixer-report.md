# Plan-Review round 2 — fixer report

Every finding in `round-2-review.md` is fixed.
`lyx loom validate-plan` over the edited plan returns `ok: true` with informational `create-new-unit` findings only.
The overview's `approved:` key is untouched.

## Changes

- **F1** — `08-record-batch-audit-dispositions.md`'s failing-verify bullet now names its finding: a fork `fabric-reference` on an `OK` report whose card verify exits 1 on the same call (step 7's first trigger), keeping the archive, `ErrBatchFailed` and `recover-batch` assertions.
  `11-recover-failed-batch.md`'s recovery test refers to that same case by name.
- **F2** — `01-audit-severity.md`'s `ClassifyViolation` compares canonical paths, resolving links on the write path and each told directory (nearest existing ancestor through `filepath.EvalSymlinks`), so a write spelled through the weft's `_lyx` classes as correctness.
  Its test table gains a row with `_lyx` made a link through `fslink.CreateDirLink` and a `parent-write` spelled through the link's target.
- **F3** — `10-post-batch-failures.md` records each later-card drift finding through `recordBatchWarning` under the identity `later-card-drift:<finding text>`, via a new `postBatchInputs.Batch` field both callers fill.
  Card 10 now edits `recoverbatch.go`, uses `auditledger.go`, asserts the state.json entry in its drift test, and its ImpactSummary says so.
  `02-audit-ledger.md`'s `recordBatchWarning` and `recordRunWarning` take `class` and `detail` strings instead of an `AuditViolation`, so the drift warning shares the path.
  `14-summary-audit-warnings.md`'s section explanation now covers drift warnings as well as fork-audit findings.
- **F4** — `27-loom-gate-refusal-sweep.md` limits the card 25 pointer to `seed-missing` and the decode and invalid-state branches, and states that the done state gets no row because step 1's short-circuit is not a refusal.
- **F5** — `23-websterengine-refusal-sweep.md` names the overview's `way-forward-phrasing` shared decision instead of the range "cards 5–19".

## Deferred

None.
