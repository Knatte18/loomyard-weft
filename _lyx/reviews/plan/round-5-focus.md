---
round: 5
focus:
  - "Session-end bound in `_lyx/plan/18-pairteardown-loop.md` and `_lyx/plan/12-loop-detach-wait.md`: card 18 writes a pid-file mark naming the teardown as loop id and claims no invocation spawns a loop between the session end and `RemovePair`. Settle whether that holds for the second and third invocation after the first `deadLoopStop`, given that card 12's `deadLoopStop` retires the pid file and writes a `loop-exited` envelope under the dead loop's recorded id that the next invocation delivers. Check that the card's Intent states the bound it actually delivers and that a test row in `TestUntilStop_DeadLoopStopsLoopExited` or the teardown tests covers repeated invocations over the mark."
  - "Waiter identity in `_lyx/plan/12-loop-detach-wait.md`: a waiter matches envelope and delivered-record files by its adopted loop id. Settle whether a waiter that spawned a loop but never read that loop's pid file, with the loop having finished and its envelope delivered elsewhere while a later loop ran, can print the later loop's envelope or write `failed` through `WriteFailedStop`. Check whether the card's never-prints-another-loop's-envelope claim and the `TestUntilStop_SecondWaiterAdoptsTheLiveLoop` rows cover that path."
---

Both entries are the surviving gating-relevant areas after round 4.
The arming-refusal paths in cards 11 and 12 were settled in round 4 and need no re-review beyond what the rubric asks.
