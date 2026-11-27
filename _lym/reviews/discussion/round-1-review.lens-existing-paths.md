```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code paths that already write or read the state each Decision changes.

**Paths traced, per decision**
- D1, board writes: every board write calls `spawnSync` (`internal/boardengine/board.go:216-218`). **Breaks D5's ordering** (F4).
- D1, the pushes D1 says stay synchronous: `internal/tendercli/deps.go:148` (`SpawnStatusPush`) and `:154` (`SpawnSync`) are detached spawns, and so are `heddlecli/cli.go:173` and `pairteardown/followup.go:91`. **Premise breaks** (F7).
- D3, owed pushes and failed-push records: `pushClassified` → `classifyFailure`/`fail` records transport, deferred and teardown-lock failures; `recordPushFailure` mails once per episode (`failedpush.go:73-107`); `IsPushOwed`/`isOwedClass` read those classes (`pushanchored.go:46-73`). **Breaks** (F1).
- D3, runfsm hold: `runread.go:238-246` reads `FailedPushesOwedBy` (`failedpush.go:185-200`) into `Inputs.PushOwed`, and `runfsm.Act` waits on `ReasonPushOwed` (`act.go:119-120`). **Breaks** (F1).
- D3/D4, refused target: `Start` refuses while `RefusedPushStands` (`start.go:122-125`), and so do `Run`'s conditions (`run.go:66-68`). **Misread** (F6).
- D5, parent sync: `syncParent` stops on `ParentAhead` (`finalize.go:619-620`) and returns a plain error on a failed tips read (`finalize.go:612-614`). **Breaks** (F2).
- D5, origin reads before and after the merge: `ownerChecks` runs the reconcile (`finalize_owner.go:47-55`); `handedOverStop` reads origin before each push and returns git-transport (`finalize_owner.go:120-128`, `finalize.go:957,987,993`). **Breaks the D5 criterion** (F3).
- D5, board `done`: `markTaskDone` (`finalize.go:996`) is a board write, so it reaches `spawnSync`. **Breaks** (F4).
- D5, past-the-merge read: `FinalizePastMerge` applies only while Finalize is running or failed transient (`innerrun_owner.go:183-201`), and `runread.go:269-285` picks the origin or local parent read. **Survives** once D5's local read is added.
- D6, run lock: `recheck` exits when the run lock is held (`drive.go:854-860`); the step takes `shedrun.RunLock` (`loomcli/wiring.go:400`, `cli.go:329`); teardown waits for it and refuses other holders (`pairteardown/teardown.go:995`, `loop.go:123-128`); cleanup tries it (`battencli/cleanup.go:220`). **Breaks** (F5).
- D8: `IsPushOwed` already reads `ErrPushLockBusy` as owed (`pushanchored.go:66`), and catch-up pushes with `StatusPushLockWait` (`catchup.go:266`). **Survives.**
- D9: the clean check counts untracked paths (`verifygate.go:194-212`). **Survives** with the exception added.
- D12: `runread.go:252-266` sets `CodeTipMissing` only when the pair is absent or its local branch lacks the tip. **Survives.**
- D13: `leftoverKind` offers a landed pair every tick (`leftover.go:83-84`), and `tearDownLeftover` sends it to `pairteardown.Run` with `Remote: true` (`dobbydeps.go:524-530`). **Survives** once D13's guard is added there.

**Findings**

F1 — BLOCKING, design, D3 (decision-record.md:79) with D1.
- **Summary:** "An owed push still writes no failed-push record" is false today.
  - `pushClassified` records owed classes and mails each episode once (`failedpush.go:73-107`, `pushanchored.go:46-52`). That is issue #238's mail.
  - `runfsm.Act` then holds every run that record names on `ReasonPushOwed` (`act.go:119-120`, `runread.go:238-246`).
  - The pusher pushes through the same `pushClassified`. As written, the record leaves both the per-episode mail and the push_owed hold in force, so an outage still stops a run's start and resume.
- **Fix:** The record must state what replaces the owed-class records and the notifier call for pusher pushes, and what `runfsm`'s push_owed hold reads afterwards.

F2 — BLOCKING, design, D5 (decision-record.md:95).
- **Summary:** D5 does not handle a local parent ahead of origin.
  - Once landing A's parent push is owed, the local parent is ahead of origin.
  - Landing B on the same parent then stops Stuck at `syncParent`'s `ParentAhead` case, whose way forward is "push or reconcile" (`finalize.go:619-620`).
  - D5 covers only the lagging and diverged cases. Its "other runs keep committing" and the Goal's "never fails on a transient error" therefore fail for the second landing.
  - Separately, the lagging check's fetched tips read returns a plain error when origin is unreachable (`finalize.go:612-614`). The record does not say how that read is classed.

F3 — BLOCKING, design, D5 and D6 against the D5 criterion (decision-record.md:228).
- **Summary:** A local landing with origin unreachable cannot return done.
  - Before the merge, `ownerChecks` runs the reconcile, and D6 turns its unreachable result into a git-transport wait (`finalize_owner.go:47-55`).
  - Around each push, `handedOverStop` reads origin and returns a git-transport stop (`finalize_owner.go:120-128`, called at `finalize.go:957`, `987`, `993`).
  - The landing therefore waits instead of finishing, which contradicts the Goal and the criterion "returns done while origin is unreachable".

F4 — BLOCKING, design, D5 (decision-record.md:98) with D1 (decision-record.md:42-43).
- **Summary:** The board `done` can reach origin before the parent push.
  - `markTaskDone` is a board write (`finalize.go:996`), and every board write spawns the board sync (`board.go:216-218`). Under D1 that wakes the board target, which pushes every owed board commit, `done` included.
  - So the board target can put `done` on origin before the landing target's parent push lands.
  - D5's promise that the landing target pushes `done` only after the parent has no mechanism: nothing holds `done` back from the board target.

F5 — MEDIUM, design, D6 (decision-record.md:112-114).
- **Summary:** "While it waits the driver holds its run lock" conflicts with how the run lock is used today.
  - Today the step takes `shedrun.RunLock` (`loomcli/wiring.go:400`).
  - The driver's `recheck` reads a held run lock as another process and exits (`drive.go:854-860`).
  - Teardown waits for the run lock and refuses any other holder (`pairteardown/teardown.go:995`, `loop.go:123-128`), and cleanup tries it (`battencli/cleanup.go:220`).
  - A driver holding it between steps would make its own re-step and recheck see "running elsewhere", and would block teardown and cleanup.
- **Fix:** The record must name which lock tells a waiting driver from a dead one. That is likely `shedrun.DriverLock`, which `leftoverKind` already reads (`leftover.go:72`).

F6 — MEDIUM, design, D3 (decision-record.md:70,73,76) and D4 (decision-record.md:86).
- **Summary:** A refused target falls out of every signal while its commits are still only local.
  - It leaves the ledger, and no pusher starts while its refused-push record stands (`start.go:122-125`, `run.go:66-68`).
  - The bar's minutes can read 0, and the all-clear can post "every branch is pushed".
  - The record does not say whether the dobby's ahead check writes the entry back each tick, which would contradict "leaves the ledger", or skips refused targets.

F7 — MEDIUM, design, D1 (decision-record.md:47-48).
- **Summary:** D1 sorts pushes into moved and kept by naming callers, and one premise is wrong.
  - It says `internal/tendercli/deps.go`'s board pushes stay synchronous because a caller reads their result.
  - They are detached spawns (`deps.go:148` `SpawnStatusPush`, `deps.go:154` `SpawnSync`) whose result no caller reads.
  - Other detached spawns, such as `heddlecli/cli.go:173` and `pairteardown/followup.go:91`, fall under neither list.
- **Fix:** State the rule by kind — every detached push spawn goes through the pusher, and only the named synchronous pushes stay — rather than by caller.

**Tool calls**
1. Grepped `isOwedClass`, `FailedPushesOwedBy`, the pushnotifier head and `runfsm` PushOwed: the context did not show whether owed pushes write records today.
2. Grepped `LeftoverLanded`, `pairteardown` Remote and `FinalizePastMerge`: needed D13's and D5's resume readers.
3. Grepped the notifier conditions in `failedpush.go` and the `runfsm.Act` push_owed hold: needed the mail and hold paths of owed records.
4. Read the `leftoverKind` rule and the teardown run-lock wait: needed D13's dobby path and D6's run-lock readers.
