---
round: 2
focus:
  - "internal/gatefixer/await.go and every constructor of gatefixer.Deps, webster's awaitFixerStrand, FixVerifyAwait and FixPlanAwait in internal/websterengine/verifyfixer.go included: does each caller give goneStrandReason the shuttle config and an absolute anchor root, so that reading a gone fixer's run record never depends on the process cwd (PATTERN-cwd-resolution)?"
  - "internal/gatefixer/rowgate.go, for a driver restarted onto a fixer record left live: follow every path that can meet that record (the fixer entry's Gate, its Final closure reached through the shuttle poll loop's expiry probe in internal/shuttleengine/wait.go, and an earlier non-fixer entry failing over it). Is an interrupted fixer replaced within its bound on each, and do the tests drive those paths?"
  - "internal/shedadapters/bouncer.go judgeCall: for each way a judge run can end with no verdict, a start error returned by shuttle's RunGated included, does the outcome spend bounce budget, and does that match the decision record's amended board entry?"
  - "internal/gatefixer and internal/shuttleengine/rundir.go: can an interrupted run's record be removed by the orphan sweep before the fixer wait reads it, and is a failed replacement spawn reported to the writer and the log?"
---
