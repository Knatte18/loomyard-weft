---
round: 2
focus:
  - "internal/runproc/end.go, planRecords and runUnits: settle whether a process in a signalled group other than the recorded leader is watched, escalated to SIGKILL after the bound and reported in the SurvivorsError when it survives. Cover a leader that exits on SIGTERM while a member ignores it, and a leader that is a zombie or absent after its group got SIGTERM."
  - "internal/runproc/end.go, runUnits: settle whether a unit whose signal call returned an error still has its members watched, so a process alive after the bound is reported rather than the End call returning success."
  - "PATTERN.md against main: settle whether every entry outside the three declared changes (PATTERN-run-process-owner, PATTERN-pair-teardown, PATTERN-leaf-packages) carries the text main carries, and whether the declared texts are reflected consistently between the overview and the pattern/ background files."
---
