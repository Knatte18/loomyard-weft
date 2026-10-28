```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fixer's two test additions (F1, F2) for agreement with their surroundings.

**LOW** — `internal/beamcli/dobbydeps_integration_test.go`, the header comment (line 3) and the test name `TestHubDobbyDeps_PrimeTidyShuttle`.
- The fix added `TaskLimit` assertions to this test, but the header and name still describe only the `PrimeTidyShuttle` seam.
- The header says the file "pins the seams hubDobbyDeps fills differently for a beam's dobby and the prime's".
- `TaskLimit` is filled the same way for both, so the new assertions fall outside that description.
- A reader choosing a file or test by name or header would not find the `TaskLimit` pin.
- The review's `fix` key asked for the assertion "in the existing integration test", so adding it there is correct.
- The header could be widened to cover both, for example "pins the seams hubDobbyDeps fills, which differ between a beam's dobby and the prime's, and the ones it fills for both".

No other inconsistency found:
- The F2 subtest reuses the `near` and `free` values and the log wording of its sibling subtests.
- The log expectation matches `taskLimitHolds` in `starts.go:123`, and the subtest passed in the gate run.
- The new subtest sits beside the other subtests of `TestStartReady_TaskLimitHold`.
- F1 and F2 touch no other test or doc.

Tool calls made: none; my context already held every changed file.
