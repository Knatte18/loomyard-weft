```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. For each guard, refusal, branch or test row a card adds, I checked whether a real entry point can reach it and whether the guarded site recurs on an unguarded path.

**Per guard, reachable or not**
- Card 2 `FinishCancel` with no `Cancel`: a fail-loud wiring guard, exempt from the reach test (`refusal-spec.md`).
- Card 7, step 1 (`Verdict` refusal): reachable, but the row the card names feeds a hand-over that `judgeOrigin` already refuses. The input that really reaches it is a board-`done` slug with no batten record. Finding F5.
- Card 7, step 2 (kept-finish re-run refusal): reachable. `lym batten cancel <slug>` without `--remove` over a kept cancel gets past `judgeOrigin`, the reached-parent check and the act rule (`Cancel` and `Pause` set aside).
- Card 7, step 3 (mid-merge or failed intent commit): reachable, by a records-side merge in progress or an I/O failure.
- Card 7 `cancelledRunRefusal`:
  - In `checkRunStatus` and `battenPreStepUnmarked`: card 10's `refuseByAct` refuses a kept cancelled record first. It is still reachable through the crash leftover (task branch gone from origin, which derives `queued` or `none`, so the act rule answers act).
  - In `DobbyTasks`: reachable when a kept cancel is re-released, or through the crash leftover.
- Card 8 `cancelMarkerRefusal`:
  - Live writer: reachable.
  - Interrupted (an end-only cancel leaves its intent unfinished): reachable.
  - The cancel window before the intent exists is not guarded. Finding F1.
- Card 8 `registeredBattenAlive` row (running record with an unfinished intent): reachable during a cancel.
- Card 9 cleanup cancelled cases:
  - Pair here, only the task branch on origin, neither, and a failed probe: all reachable. `TaskBranchOnOrigin` runs `ls-remote`, so it fails offline.
  - Nil `PairStands` default: not reachable in production. Finding F6.
- Card 9 `Teardown.Run` → `removeCancelledRecord`: reachable from `lym fabric remove`. Run also serves the origin-reconcile teardown (`originseams.go:56`) and `beamfork`, so the deletion covers those paths too.
- Card 10 `refuseByAct`: reachable in `battenPreRun`, `battenPreStepUnmarked` and `startFirstPass` for the covered, hand-over, terminal and read-error cases.
- Card 11 `planPass` act gate: reachable. Its keep of `drivable`'s paused-with-work-on-parent clause leaves dead code. Finding F2.
- Card 11's remembered owned-elsewhere branch stays reachable. The dobby pulls the Bolt on probe ticks (`dobbyengine/loop.go:66`, `handover.go:19-22`), so the local Bolt does not go stale.
- Card 12 `refuseByAct` (cancelling, terminal-cancelled and read-error rows): reachable. `lym loom drive` is spawned only by start and resume, so leaving it unguarded is consistent.
- Card 14:
  - Pause owner refusal: reachable.
  - `Record` and `Clear` errors: reachable.
  - Resume's terminal wait: reachable, but it strands a slug pause entry. Finding F4.
- Card 3 `Refusal` for paused: reachable, but for an intent-derived pause it names a verb that refuses again. Finding F3.

**Findings**

**F1 — MEDIUM, design. Cards 8 and 7 (step 3): a cancel already under way reads as no cancel until its intent is committed.**
- Card 8 makes every reader treat "present" as "a cancel intent without a finish", and `takeCancelMarker` now takes only the lock.
- Card 7 writes and commits the intent only after the lock is taken and the Windows check passes.
- In that window the start guard, the pre-drive check and `DobbyRuns.CancelMarker` all read no cancel. A `lym batten run` start or seed can then proceed while the cancel ends it.
- Today the marker is written inside the lock take (`internal/battencli/cancelmarker.go:137-159`).
- Fix: readers count a cancel lock another process holds as a live cancel, whether or not an intent exists.

**F2 — MEDIUM, design. Card 11 keeps a branch whose state can no longer give a pass.**
- `drivable` still admits "paused and `ReachedParent`" (`internal/dobbyengine/passes.go:69-73`), and `DobbyRuns` still reads `ReachedParent` for every paused run (`internal/battencli/dobbyreads.go:203-216`).
- The dobby sets nothing aside, so `Derive` reads a batten shed `paused` as `Paused`, and the act rule waits it (card 3; Shared Decision paused-before-landing).
- `planPass` therefore never passes such a run. The clause, the `RunView.ReachedParent` field and its per-tick read are dead.
- Card 11 also edits `TestDobbyRuns_ReadsWhetherAPausedRunsWorkReachedItsParent` without saying the clause goes.
- Fix: drop the clause, the field and the read, together with that test's assertions.

**F3 — MEDIUM, consistency. Card 3's `ReasonPaused` refusal names a verb that cannot lift a pause that comes from a pause intent.**
- Its way forward is `lym batten run <slug>`, which sets only `ShedPause` aside (cards 3, 10). A `Paused` derived from the record's pause intent therefore stays `Paused`, and the verb refuses again with the same text.
- Reachable input: a pause intent in the batten record with no pause entry on this machine. That happens after `.lym` is removed (D3 makes the pause record a cache), on a fresh machine, or on an adopter before D6's catch-up.
- `PauseCovers`, which names `lym tender resume`, fires only when a local entry exists.
- Fix: word an intent-derived `Paused` with `lym tender resume <slug>`, and keep `lym batten run` for a shed pause.

**F4 — MEDIUM, design. Card 14: `lym tender resume <slug>` refuses to lift a slug entry of a landed or cancelled run.**
- `Resume` asks `Verdict` with only `Pause`, `Cancel`, `ShedPause` and `PushOwed` set aside. A `Landed` slug (board `done`, no record) or a kept `Cancelled` record still waits as terminal, so the entry is never lifted.
- Card 14's `Pause` asks no verdict, so `lym tender pause <slug>` on a done or cancelled slug writes that entry.
- The `Landed` way forward, `lym board list`, lifts nothing. Only a hub-wide `lym tender resume` or `lym batten cancel` (`internal/battencli/cancelcmd.go:488-509`) does.
- Fix: the resume also sets a terminal state aside, since lifting an entry drives nothing, or the pause refuses a terminal slug the same way.

**F5 — LOW, consistency. Card 7 step 1's named test input is rejected upstream.**
- The card's row for the act-rule refusal feeds a hand-over verdict.
- `judgeOrigin` already refuses an owned-elsewhere slug from the same local-Bolt read whenever the task worktree is absent or origin is unreachable (`internal/battencli/cancel.go:553-560`). With the worktree present, the reconcile's `EntryOwnedElsewhere` refuses it (`cancel.go:571-572`).
- A batten `done` record at Worktree-Teardown is refused by the reached-parent check (`cancel.go:779-781`).
- The input that does reach the branch is a slug with board status `done` and no batten record (terminal `Landed`), and no row names it.
- Fix: add that row.

**F6 — LOW, design. Card 9 adds a branch for a state production never has.**
- A nil `PairStands` "reads as a pair standing here". The card itself says only a hand-built `cleanupDeps` has a nil one, and `fabricCleanupDeps` always fills it.
- The branch serves test construction only, which the code-quality rule against handling states that cannot happen rules out.
- Fix: drop the default and have every test-built `cleanupDeps` set `PairStands`.

**Tool calls**
1. Grep of `RemovePair` and `Teardown.Run` callers, the dobby's owner recheck and Bolt pull, and the loom verbs' owner refusals: to see whether the guarded sites recur on paths the cards don't cover, which the manifest didn't show.
2. `internal/battencli/wire.go:1185-1200`: which removal batten's own teardown row uses.
3. Grep of `OriginSeams` and the removal calls in pairteardown: where the reconcile seams live.
4. `internal/pairteardown/originseams.go:124-200`: whether the origin-reconcile teardown goes through `Run`, and so through card 9's hook.
