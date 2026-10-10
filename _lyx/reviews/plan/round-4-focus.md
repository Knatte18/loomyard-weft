---
round: 4
focus:
  - "Card 18 (`_lyx/plan/18-pairteardown-loop.md`) `seizeLoopLock` releases the loop lock once `EndSession` has ended the reed session, and card 12 (`_lyx/plan/12-loop-detach-wait.md`) `runUntilStop` spawns a loop when it reads the lock free with no pid file and no envelope. Settle whether an `--until-stop` invocation that was waiting on the held lock can spawn a detached loop after the session ends and before `RemovePair`, and whether card 18's stated bound and its `teardown_test.go` rows cover that case."
  - "Card 11 (`_lyx/plan/11-loop-body.md`) routes an `--until-stop` arming error through `ReportLoopArmError` when the pre-run resolved a location and a run-id, else through `ReportArmError`. In `internal/loomcli`, `resolvePersistentPreRun` calls `c.arm`, and `c.location` is assigned only at the end of `wire`. Settle whether a missing-hub-config refusal under `lyx loom step --until-stop` reaches `ReportLoopArmError` and writes `failed`, and whether a planned test drives that subtree."
  - "Card 12 waiter: settle whether a waiter compares the envelope's or delivered record's `loop_id` with its adopted id before printing, and what happens to a waiter whose adopted loop is gone while a later loop's records exist."
  - "Cards 11 and 12, arming refusal paths other than the waiter's own pre-run: settle which write order `ReportLoopArmError` uses when a dead loop's pid file is present, and where a refusal in the detached loop process's own pre-run lands, as stated in each card's `**Intent:**`."
---

The focus lists the questions round 3 left open or only partly settled.
