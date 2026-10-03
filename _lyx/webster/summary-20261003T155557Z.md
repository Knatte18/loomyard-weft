# verify-publish-hardening: stopped at batch 16 on plan drift

Batches 02, 03, 04, 05, 10, 11, 12, 13, 01, 08, 14 and 15 recorded `done`, each reporting no deviations except as noted below.

What was built:

- Gated shuttle runs no longer treat a waiting turn end as an arrival (02).
- Every claude session launches with a pointer to `prompt.md` rather than inlining the prompt (03).
- Fabric verbs stage tracked edits and list untracked files mid-merge (04); the conflict session's tracked edits land in the merge commit, untracked files halt, stale reports are cleared (05).
- `recover-batch` blocks to terminal and Merriam waits on notifications only (10); Merriam's strand role is `webster` (11).
- `begin-batch` renders each card's gate command (12); Plan-Write greps for reworded messages (13).
- `internal/verifytree` exists with the verify function, record and marker (01); Webster-Burler carries a must-pass verify gate (08).
- The verify-gate report, findings text and card hint exist (14); webster's plan-level verify is a must-pass gate on Merriam's strand (15).

Reported deviations: batch 04 added `StageTrackedChanges` to the pinned-method list in `cmd/lyx/gitrepoboundary_test.go`.
Batch 08 changed `internal/loomshed/seam_enforcement_test.go` and `internal/shedrecipe/fixture_test.go`.
Batch 15 added `Geometry.VerifyDir`, deleted `internal/websterengine/integration_test.go` and added `wiring_parentbranch_integration_test.go`.

Stop reason: `begin-batch 16` refused with `plan_drifted`.
Card 17 targets `internal/websterengine/integration_test.go#`, which batch 15 deleted, so the plan no longer matches the tree.
The operator edits card 17, runs `lyx webster rebaseline --card 17`, and resumes with `lyx webster run`.
Batches 16, 17, 06, 07, 09, 18, 19 and 20 did not run, and the plan-level verify was not run.
`gofmt -l` flags `internal/shell/posix.go` from batch 03.
