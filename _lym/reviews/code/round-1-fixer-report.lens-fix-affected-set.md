```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches for the findings whose premise is a set (F1 verify sites, F8 `checkHead` callers, F7 `exitCodeOf` callers, landing `check` callers) and check the fix covers every member.

No findings.

Members checked, all covered:
- **F1, `verifytree.Verify` callers:** these are `loomshed/verifygate.go:127`, `websterengine/verifygate.go:244` (the seam), `landingshed/verifygate.go:140` (the seam, reached through `check` and `checkHead`) and `webstercli/verify.go:72`. Each now reads `RunnerCrash`:
  - `loomshed` sets `Terminal` from it.
  - `websterengine` fails terminally on both the first run and the rerun.
  - `webstercli` returns its own error envelope.
  - `landingshed` handles it in `verdict` and `headFailureReason`.
- **F8, `checkHead` callers:** the only caller is `prreview.go:557`, and it takes the new `crashed` return.
- **Landing `check` callers:** `finalize.go:714`, `fixerrun.go:287` and `parentalign.go:198` all consume `crashed`.
- **F7, `exitCodeOf` callers:** `tierrun.go:280` and `build.go:48` share the gatecli helper, and the `doc.go` edit covers the build. `driftengine/steprun.go` has its own unrelated `exitCodeOf`.
- **Gate output readers:** the runner crash line is read through `gateslot.FindRunnerCrash` (`verifytree` and `detachfixer`), with no other consumer of the gate's output left unhandled.

Tool calls:
1. Grep for `Verify` callers, `RunnerCrash` readers and `checkHead` callers: to rerun the F1 and F8 set searches, which my context lacked.
2. Grep for landing `check` callers and `exitCodeOf` callers: to rerun the F7 search and confirm all three landing `check` callers, which my context lacked.
