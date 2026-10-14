---
round: 3
focus:
  - "internal/reedcli/attach_unix.go handOver: settle whether syscall.Exec now receives a PATH-resolved tmux binary as both its path argument and argv[0], whether a failed lookup returns exit code 1 with the error, and whether the testing.Testing() branch still runs runAttachChild with the original name."
  - "The five reedcli smoke tests TestSmokeRepaintCandidateMeasurement/Candidate1, TestSmokeRepaintCandidateMeasurement/Candidate2, TestSmokeDotFill/FloorIsCleanOnASettledAttach, TestSmokeColdStart/ColdAttachBootsThenFailsOnTheTerminalHandoverNotOnNoSession and TestSmokeLifecycle/AttachRendersInsideHarnessPane: settle whether the branch's own range leaves any other path where a bare tmux name reaches an exec without lookup."
---

Round 2's only finding was a BLOCKING exec of an unresolved `tmux` name; round 3 checks that the fix is complete and that no sibling site repeats it.
