---
round: 2
focus:
  - "internal/shedengine/run.go, the failed-step persist: for a failure whose transient class is non-empty, does the persisted state meet the first point under `## What to do` in the decision record's `## Goal`, as amended by any `Added after Discussion` entry? Where the persisted state changed, check every reader of it, including decideDrive in internal/sheddriver/driverules.go, failedHalt in internal/loomcli/halt.go, the run state machine and the board status."
  - "The texts describing the transient-stop halt: the doc comment on AnomalyTransientStop in internal/loomengine/anomaly.go, the transient-stop paragraph in internal/loomcli/doc.go and the transient-stop drop rule in contracts/stencils/friction/friction-template-reflection.md. Does each hold for every class in internal/shedengine/transient.go, given which classes networkFailure in internal/sheddriver/driverules.go re-steps and the one re-step in internal/shedverbs/loop.go?"
  - "The doc comment on ensure in internal/landingshed/parentguard.go: does what it says about a gap between wall-clock and awake time hold on every platform the repository builds for, and under a wall-clock step?"
  - "TestClassifyHalt in internal/loomengine/anomaly_test.go: is the order between build skew and transient stop that the ClassifyHalt doc comment states pinned by a case that sets both?"
---

Round 1's open points, each restated as a question for a fresh reading.
