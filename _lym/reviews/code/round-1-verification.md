---
round: 1
entries:
  - key: parallel-group-runs-global-logger-capture
    status: closed
    finding: F1
    reason: "a22a8db11 removes the row from the parallel CommandRows group and adds a serial TestBattenIntegration_CommandRowsIgnoreACommandSetInTheTasksTrackedLayer (tiergate 1, own hub, posixOnly nil guard, logcapture comment), step unchanged"
---
