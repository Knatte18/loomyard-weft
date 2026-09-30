---
verdict: BLOCKING
rationale: "All fifteen round-1 findings are fixed in the current plan, but the fixer's test insertions left card 8's failing-verify test bound to a correctness finding (The same finding now follows the FAILED-report correctness bullet), so D5's same-call policy-finding verify failure, a must-cover scenario, has no unambiguous test"
---

# Round 1 verdict — BLOCKING

## Round-1 findings: all resolved

The review was written against the pre-fix plan; the fixer's report lists a change per finding, and each change is present in the current cards.

- F1: card 2 now edits `beginbatch.go` and `recoverbatch.go` to inherit `AuditWarnings` beside `StartSHA`, and adds `TestBeginBatch_ReBeginKeepsAuditWarnings`.
- F2: card 8 replaces the vacuous second-call-on-a-terminal-batch assertion with the reachable no-report → re-begin → OK-report retry.
- F3: card 15's named-spawn end-to-end test asserts batch 2's warning appears once in `summary.md`'s `## Audit warnings`.
- F4: card 8 step 3 grounds the no-report and `FAILED`-report extension in D2 and D8, tests both, and card 11 takes the late-report half.
- F5: card 27 maps `fabric-sync` to `lyx fabric checkout` and splits `worktree-clean` into git for the warp and `lyx fabric commit` for the weft.
- F6: card 24's fixture fails the weft commit through a stale `index.lock`.
- F7: card 26 adds `commit-records`' commit and push failures, groups approve's lookup failures, and states that `validate-*` verdicts are not refusals; card 28 says the same.
- F8: card 27 groups the `CheckSeed` verdicts pre-empted by Shed's step-1 gate and keeps a test only for the out-of-vocabulary-outcome branch.
- F9: card 22 gains the `loom-status-spec.md` Prosa target; card 17 updates `appendHistory`'s vocabulary comment.
- F10: card 8 step 7 also re-runs verifies when the record already carries `AuditWarnings`.
- F11: card 1 uses `filepath.Dir(geom.WebsterDir)` and states the per-spawn `named-spawn` split.
- F12: card 7 edits `state.go` for the status comments and its ImpactSummary states each reader's treatment of `failed`.
- F13: card 23 puts the unknown-batch way forward in `findBatch`.
- F14: card 4 adds `TestRecoverSpawn_RecordsCardSet` in `recoverbatch_test.go`.
- F15: card 26 is split into `26-loom-cli-refusal-sweep` and `27-loom-gate-refusal-sweep`.

I ratify every one of the fifteen as a legitimate finding under the rubric (fidelity, verify coverage of must-cover scenarios, `ImpactSummary` accuracy, granularity).
None re-derives a gate check or a dependency edge.
The review carries no `## Focus departures` section.

## Remaining finding

**Card 8's failing-verify test no longer tests D5.**
In `_lyx/plan/08-record-batch-audit-dispositions.md`, the test bullets now run:

- "A correctness finding with a `FAILED` report → batch terminal `failed`, the report archived."
- "The same finding with a card verify that exits 1 → batch terminal `failed`, the report file moved to its archive name, …"

"The same finding" now refers to a correctness finding, which fails the batch in step 3 whatever the verify does.
Read as written, the test passes with step 7's verify re-run deleted.
The acceptance criteria's second must-cover scenario needs a policy finding (`fabric-reference`) on an `OK` report whose card verify fails, on the same call: batch failed, report archived, `recover-batch` proceeds.
The retry bullet's "same flow with a verify that exits 1" exercises only step 7's second trigger (warnings recorded on an earlier no-report call), and it asserts no archive.
Card 11's "card 8's failing-verify failure" inherits the same ambiguity.

Fix: name the finding in that bullet — a fork `fabric-reference` on an `OK` report with a card verify that exits 1 — and keep its archive, `ErrBatchFailed` and `recover-batch` assertions.
Card 11's recovery test then refers to that case.

## Checked without a finding

- Every decision D1–D12 and every listed constraint maps to a card; cards 13 and 16 are licensed by D6/D7's in-session way forward and by the acceptance criteria's regression requirement.
- The remaining must-cover scenarios each map to a card test with the take-the-way-forward shape.
- `## verify:` runs `go test ./...` and `go test -tags integration ./...`, covering every targeted package and the integration-tagged tests; `smoke` is compiled through `go vet`, which the rubric exempts.
- No `Custom` card exists.
