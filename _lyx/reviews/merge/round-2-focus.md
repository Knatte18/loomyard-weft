---
round: 2
focus:
  - "In internal/battenshed, check every failure log in the binary re-step check (innerrun_binaryrestep.go: restepSkipped, restepRecorded) and in publishRetryDecision (publishretry.go, the Go verify-retry records read): does a failure that persists across polls still log Warn more than once per episode, and does each seam log at recovery? Judge against PATTERN-log-levels and the seamFailed/seamRecovered episode guard over innerRunProducer.failures."
  - "In innerrun_binaryrestep.go, check that the dirty-pair skip line does not log at Info on every due check while the halt, build and dirty-path list are unchanged, and that any remembered state resets on a new stop key and a changed build."
---
