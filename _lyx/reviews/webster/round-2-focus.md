---
round: 2
focus:
  - "internal/shuttleengine/wait.go and send.go: `Run.Wait` holds a gated Done that arrives while a pause covers the session in `boundaryHeld`. Settle whether `boundaryHeld` is cleared everywhere `gateAtBoundary` is set false and in the arrival branch before it sets `gateAtBoundary` true, so a gated Done that arrives covered, a new writer turn, then a second gated Done that arrives uncovered evaluates the boundary exactly once, and whether a gate_test.go row pins that sequence."
  - "internal/proc/procgroup_linux.go `configurePlatformGroupTerminate` and procgroup.go `ConfigureGroupTerminate` doc: settle whether a group SIGTERM failing with an error other than ESRCH now kills the command alone so the shell cannot outlive the cancel, whether ESRCH still returns its error, whether the doc line matches the behavior, and whether a Linux test in internal/proc pins the failed-SIGTERM branch and `awaitGone` called without a cancel."
  - "PATTERN.md overview lines for `PATTERN-review-round`, `PATTERN-fabric-vocabulary` and `PATTERN-documentation-lifecycle`: settle whether the overview still states each rule's essential clause against its pattern/ background, and whether the headroom lint stays green."
---

Focus targets the fixes to the round-1 findings, which no fresh reviewer has seen.
