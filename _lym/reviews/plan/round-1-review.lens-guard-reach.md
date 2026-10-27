```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. I traced each guard, refusal, branch and test row the cards add, and checked whether it can be reached.

**Per guard, reachable or not**
- Card 2, `BuildShed` refusal on a failing machine name: reachable, but only on a continue pass or with no owner seams. `battenPreRun` returns before `preflightRunOwner` when `c.continuePass` (internal/battencli/arm.go:731-735). A fresh run or a full pass is refused upstream by `thisMachine` (owner.go:120, reached from arm.go:400 and arm.go:737). The empty-name half cannot occur (see F4).
- Card 3, `machineLandings` failing name read: reachable. Empty name: cannot occur in production (F4).
- Card 4, each `countLandings` branch (first read, tidy in flight, no flag, flag set): reachable through the tidy harness.
- Card 5, `CheckWith`'s `beforeExec` and `failed` callbacks: reachable; a nil callback is skipped.
- Card 6, `AppendLog` into a missing directory: reachable (the card 8 exit and the card 10 refusals). `RotatePastBound` on an absent file: reachable.
- Card 6, `rotateOutBeforeSpawn`'s held-lock branch on Windows: reachable only by a race. `ensureOne` calls `Spawn` only after `Live` read the lock free (internal/dobbyengine/start.go:229-237). The test reaches it through the injected goos and a test-held lock; Decision 5 licenses it.
- Card 7, step 1 re-exec read error: reachable. A directory named `state.json` makes `os.ReadFile` fail with an error other than not-exist (internal/state/state.go:115-120).
- Card 7, step 3 refused fresh start, refused re-exec (own pid on the second read), and refused re-exec with another pid on the second read: all reachable through a held lock and the rewriting goroutine.
- Card 7, step 5 fatal error after a re-exec (a `Locked` error): reachable.
- Card 7, `recordStart`'s three cases and `tick`'s refill with ticks: reachable.
- Card 8, the exit test after each tick, before `CheckBinary`: reachable and tested.
- Card 8, the exit test at the top of each iteration (the watch-event and probe-free path): reachable in production, but no test row reaches it (F1).
- Card 8, the stop of the kept tidy, the failed stop and the no-tidy case: reachable.
- Card 8, the prime dobby never taking the exit: reachable.
- Card 9, change lines, episode lines, the heartbeat on its interval and the heartbeat's runs-unreadable line: reachable.
- Card 10, step 1 refusal line and notice on a task pair with a `.lym/beam` directory: reachable, but the row cannot occur in the planned test sequence (F2).
- Card 10, refusal with no resolved location: reachable.
- Card 10, step 3 notice after a wiring failure: reachable through `PauseGeometry` or `noticeProbeInterval` failing. When `hubDobbyDeps` fails at its first step, `lymcwd.ResolveWorktree(geom.WorktreeRoot)` (internal/beamcli/dobbydeps.go:71), the card's own re-resolve of the same root fails too, so only the line is written. That is the card's stated bound.
- Card 10, step 4 rotation of `dobby.out` under the lock: reachable in the tier1 test.
- Card 10, step 2 failed reopen: a raw I/O failure, exempt.
- Card 11, `seedAdoptGate` with no `finalize` or with `adopt` already held: reachable.
- Card 12, `Started` for a run with no slots, `Ended` for a carried tier, and `Ended` for a blocked tier: reachable. `report`'s refusal branch after `Started` is a site with no `Ended` (F3).
- Card 13, check lock held: reachable. Reconcile failure: reachable.
- Card 13, "no tier ended": reachable on a refused key. Its "busy slots" case cannot occur (F5).
- Card 13, absent board-level lym dir as an error: a fail-loud wiring guard, exempt.
- Card 14, `ensureCheck`'s alive skip, dead record, green record and failed spawn: reachable.
- Card 15, `ended without a verdict` for a dead pid or differing ticks, and no record: reachable.

**Findings**

**F1 — MEDIUM, design, card 8 (`TestRun_WorktreeGoneExits`): no row reaches the exit test at the top of the loop.**
Every row removes the worktree inside a tick through the tick hook. The new exit test after each tick, before `CheckBinary`, then ends the loop before it sleeps. So the "one watch event, with no probe due on the next iteration" never reaches the top-of-loop test. Neither does the acceptance criterion "a beam dobby woken by a file event, not a probe tick, after its worktree is removed still exits". The suite would stay green if the top-of-loop check were deleted.
Evidence: card 8, `_lym/plan/08-dobby-worktree-gone.md`, the Intent's loop paragraph and the test setup ("the tick hook removes the whole worktree on tick 1 and sends one watch event"); the `Tick` hook runs last in `tick` (internal/dobbyengine/loop.go:304-309).
Fix: add a row that removes the worktree after a tick ends, for example in the `Sleep` seam, and wakes the loop by a watch event with no probe due. Assert the exit and the tidy stop.

**F2 — MEDIUM, design, card 10 (`TestRunDobbyWith_TracesEachExitBeforeRun`): an early row breaks the hub the later rows need.**
The rows run in sequence over one hub. The third row's `wire` fake "removes the prime's worktree link". The prime is the hub's main worktree, and the task pairs are git worktrees added from it (internal/hubforge/hub.go:214, `fabricengine.WorktreePath`). After that row, `lymcwd.ResolveWorktree` and `fabricengine.PrimeName` (internal/fabricengine/worktreelist.go:147, which runs git `List`) fail for the task pair. The later row, "task pair … refused as neither prime nor beam, gets the start-refused line in its own `dobby.log`", then refuses at the resolve with a nil location, so its state cannot occur.
Fix: run the link-removal row last, or on its own hub, or have the fake restore the link before the next row.

**F3 — LOW, consistency, cards 12 and 13: a tier that started can end the call without `Ended`.**
`report` returns on `result.refusal` before writing a record (internal/gatecli/run.go:448-451), and card 12 calls `Ended` only past that point. A tier whose body refuses after `Started` therefore never calls `Ended`. Card 13's `Ended` is the only step that clears `Running`, and step 5 ends the record `failed` without clearing it. The final `check.json` then shows a tier still in flight. Card 15's view of a failed record would render that stale tier.
Fix: card 12 calls `Ended` (or a refusal callback) on the refusal branch, or card 13's step 5 clears `Running` when it ends the record.

**F4 — LOW, consistency, cards 2 and 3: the "returns an empty name" branch cannot occur in production.**
`machinename.Name` never returns an empty name without an error, since `FromHostname` refuses one (internal/machinename/machinename.go:21-26). The empty-name rows of `TestMachineLandings`, and card 2's "or returns an empty name", guard a state the production seam cannot produce. Only the fakes reach it.
Fix: drop the empty-name case, or name it as a seam-contract guard.

**F5 — NIT, consistency, card 13 step 5: "busy slots" cannot end the gate before a tier.**
`RunGate` sets `--wait`, and with `wait` set `acquireSlot` blocks until the slots free and never returns `SlotBusyExit` (internal/gatecli/run.go `acquireSlot`, `waitCtx` bounded only without `request.wait`). The no-tier-ended branch is still reached by a refused key.
Fix: drop "or busy slots" from the step.

**F6 — LOW, design, card 8: the record read after each tick can recreate the removed path.**
After the exit test passes, the loop reads the record with `ReadState` to refresh `keptTidy`. `ReadState` goes through `state.ReadJSON`, which calls `os.MkdirAll` on the record's directory (internal/state/state.go:96). A removal between the test and that read recreates the `.lym/beam` stub. That is the same recreation the decision's bound names for a tick in flight, but at a site the card adds and the bound does not cover.
Fix: read the record there with `state.ReadJSONNoMkdir` after a stat, or name the window in the doc.go bounds.

**Tool calls**
1. `grep`/`sed` of `internal/battencli/arm.go` (`battenPreRun`, `battenPreStep`) and the owner-check callers: the manifest lacked the pre-run order that decides whether card 2's refusal is reached upstream.
2. `grep` of `refuseContinuePass` and `ownerPreflight` in `internal/battencli/owner.go`: to confirm a continue pass makes no machine read before `BuildShed`.
3. `grep` of `internal/hubforge/hub.go` (`PrimeWorktree`, `PairCodeWorktree`) and the `worktree add` sites: to confirm the prime is the main worktree the pairs hang from, for F2.
