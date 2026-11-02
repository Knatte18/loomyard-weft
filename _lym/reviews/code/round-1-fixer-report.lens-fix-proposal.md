**Findings**

1. **MEDIUM — F1: `internal/loomcli/pawlwiring_test.go` `TestSurfaceBeamUpWarnings` does not pin the review's stated regressions.**
   - The review's concern was that reverting the Beam-Up step to `_, err := orchcli.BeamUp(...)` fails no test, nor does a `surfaceBeamUpWarnings` that returns the post error.
   - The step closure sits inside `pawlDeps` and is still untested, so that revert still passes every test.
   - `surfaceBeamUpWarnings` has no return value, so the "failed post" row only proves the function returns normally.
   - A variant that propagated the error could not compile against the test, but a step that never calls `surfaceBeamUpWarnings` still passes.
   - The proposal pins the helper, not the wiring.
   - Evidence: `pawlwiring.go` calls `surfaceBeamUpWarnings(notify, slug, report.Warnings)` only inside `Steps[0].Run`; the test calls the helper directly.
   - A sound fix would let the test run `pawlDeps(...).Steps[0].Run` with an injectable `BeamUp`.

2. **LOW — F3: `smoke_drivingsurface_test.go` new subtest "start beside a runs strand adds the status strand in its own window" is invisible in the fixer's gate runs.**
   - It is a tier-2 test, and the tier-2 gate run reported 240 skipped of 275 with no sign whether it ran.
   - It can detect the dropped `strands` argument only if reed refuses the second overview strand, so that `ensureStatusStrand` returns an error. I could not confirm that refusal from my context.
   - If reed instead accepted a second overview strand, the status strand would land in the runs window, and the test's window-inequality assertion would catch it.
   - Either way the test is sound, but the parent should confirm it actually ran.

3. **No findings** for F2 and F4.
   - The F2 rows put `runs` stale and `loom-status` live, in both orders, so a reordered `runs != nil` / `loomStatus != nil` switch in `ensureRunsStrand` fails them.
   - The F4 doc edits match the wording in `internal/loomcli/doc.go` and `internal/orchcli/doc.go`.

Tool calls: none; my context already held everything this lens needed.
