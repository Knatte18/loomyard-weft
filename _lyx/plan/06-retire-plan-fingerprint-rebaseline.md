# Card 6 — retire-plan-fingerprint-rebaseline

**Delete:**
- `internal/websterengine/fingerprint_rebaseline_test.go`

**Edit:**
- `internal/websterengine/fingerprint.go`
- `internal/websterengine/render.go`
- `internal/websterengine/runlevel_test.go`

**Uses:**
- `plan:internal/websterengine#ArchiveRunRecord`

**Intent:**
Remove `RebaselinePlanFingerprint`, whose only production caller was PR-Rework before card 5 (D8), and replace the resume-over-an-extended-plan test with the fresh-run behaviour a rework generation now gets.

- Delete `RebaselinePlanFingerprint` and its tier-2 test file.
  `restampAndSaveFingerprint` stays: `Run` calls it.
  `lyx webster rebaseline` (`Rebaseline`) and `restore-plan` are untouched.
- `RenderRemaining`'s doc comment stops citing PR-Rework appending cards; it serves crash-resume within one generation.
- `runlevel_test.go`'s `TestRun_ResumesOverExtendedPlanAfterRebaseline` is replaced by a test over the same run fixture: a finished run is archived with `ArchiveRunRecord`, the plan is replaced by a generation whose `first_card` is `3`, and `Run` starts a fresh run whose state records only the new generation's batches and whose Master prompt names only those batches, with no `ErrFingerprintMismatch`.

**ImpactSummary:** one exported function with no remaining production caller is removed, with its own tests; `Run`'s fingerprint path and the operator rebaseline verbs are unchanged.
