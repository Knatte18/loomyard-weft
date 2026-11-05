```yaml
lens: guard-reach
outcome: reported
```

Task: guard-reach lens over cards 1–18, checking for each guard whether it can be reached and whether the guarded site recurs on a path no card guards.

**Per guard, reachable or not**

- Card 1, retired-row move at step 2 (`findProducer` miss with a `Retired` hit): reachable. A new-binary `lym loom resume` or re-step of a status naming Parent-Align, Beam-Describe or PR-Review gets there, and no earlier check rejects the row. The only unknown-row refusal is `run.go:249`.
- Card 1, `RetiredMove` error leaves the status unchanged: reachable (fake seam, and in real use a failed commit or notice step).
- Card 7, fork refusal for a member under a ready beam: reachable. A hand `lym batten run` while the beam waits at Beam-Await with the ready record standing reaches it.
- Card 7, `ready` read error in the fork: reachable (corrupt record).
- Card 7, dobby `holdReadyBeam`: reachable. With `BeamReady` nil on the prime it holds nothing.
- Card 8, each failing step of `MoveOffRetired`: reachable. A GitHub read failure carries the pull request; a closed or merged scratch pull request is not carried.
- Card 8, ending the live PR-Review session: reachable. The site recurs unguarded for Parent-Align (finding 4).
- Card 9, re-stepping an interrupted Beam-Await stop: reachable, since a loop exit leaves the status running with kind interrupted. The stamp-gap restart and the `CompletedWake` error branch are reachable.
- Card 10, entry detection by the board hold naming Finalize: reachable after a crash between accept and Done while the status stays running. The 3b resume write does not fire, so the board keeps Finalize. A resume from a non-running status rewrites the board first (finding 2).
- Card 10, entry detection by the ready record standing with the await marker absent: reachable, via a goto from a halted Finalize, or a resume after the accept's removals completed.
- Card 10, hold standing with no ready record (lift-hold branch): practically unreachable, since only Finalize, Beam-Down and the retired-row move remove the record. It stays as a defensive branch and is not reported.
- Cards 5 and 10, test rows "hold named PR-Review": not reachable (finding 3).
- Card 10, `ReadyStands` error: reachable.
- Card 11 halts (closed, pending past the window, `RefusalOther`, `RefusalMethod`), the merged-at-another-head branches and the scratch-record merge: all reachable. A pull request closed or merged on GitHub after ready is an input the earlier `lym beam ready` check cannot prevent.
- Card 12 `judgeAwaitingRun`: accepting a blocked Beam-Await is not reachable, and a failed one is wrongly refused (finding 1). Its other refusals (no run, not at Beam-Await, done) are reachable.
- Card 12 `judgeNotReady`: reachable between `lym beam ready` and the accept.
- Card 13 `prOpen` refusals (no stamped description, no commits, each missing origin fact) and the already, reopened and merged branches: reachable. The record-write-after-create failure is reachable through the fake.
- Card 13 `prUpdate` with no record: reachable.
- Card 14 `prStatus` with none recorded at Beam-Await or at Finalize: reachable.
- Card 14 `prClose` (none recorded, ready standing, failed mode read): reachable.
- Card 15 refusals 1–6: reachable through the fake seams. Refusal 1 inherits finding 1. Refusal 4's validation finding is reachable only through a hand edit of `summary.md` after `lym beam describe`, but it is reachable.
- Card 16 `announceLandings` branches: reachable.
- Card 17 `ownLandingHold` (Finalize, Beam-Down, a retired row an old binary left): reachable.

**Findings**

1. **MEDIUM, design** — card 12 `judgeAwaitingRun`, reused by cards 13–15.
   - The card passes a Beam-Await run that is "running, blocked or paused". Beam-Await never halts blocked: `BeamAwait.Call` returns only Done, Yield or an error (`internal/pawlengine/beamawait.go:71-94`).
   - So the blocked case cannot occur, and card 12's test row "one blocked are described" pins a state no run reaches.
   - The halt Beam-Await can reach is an error stop. If, as I believe, a producer error persists the run as failed (a state `run.go:231` names), the card refuses it. D3 and D5 say "running or halted" (`decision-record.md:54`, `:86`), so a halted Beam-Await after an error stop would be refused by describe, the pull-request verbs and ready.
   - I did not confirm in `stepLocked`'s error path that a producer error persists as failed.

2. **LOW, design** — card 10's entry rule.
   - The scenario: a crash inside `awaitRun.accept` after the land request is removed and before the await marker is (`beamawait.go:306-310`), followed by a resume from a non-running status or a goto.
   - The 3b resume write commits the board as Beam-Await (`run.go:317-318`; `shed.go:31` says persist calls CommitStatus). Neither of card 10's two detections then fires: the board no longer holds, and the await marker still stands.
   - The run takes today's path and waits for a land request. The dobby, with its land episode recorded (`internal/dobbyengine/land.go:50`), never writes one.
   - Recovery exists only through the dobby-down notice that `noticeDownDobby` posts (`lym beam start`, then a hand `lym beam land`). The guarded site recurs outside card 10's guard.

3. **LOW, consistency** — the "hold named PR-Review" rows in card 5 (`TestBeamAwait`) and card 10.
   - The old accept wrote only Parent-Align (`beamawait.go:295`).
   - A status at PR-Review or Beam-Describe is moved by card 1 before Beam-Await runs.
   - Every transition into Beam-Await rewrites the board through CommitStatus. Goto calls none itself, but the next step's write does (`internal/shedengine/goto.go:67`, `run.go:317`).
   - So no real path enters Beam-Await with the board naming PR-Review or Beam-Describe. Parent-Align is the only reachable retired-name hold, and those rows test a state that cannot occur.

4. **LOW, design** — card 8, `EndSession`.
   - It ends only a live PR-Review session.
   - Parent-Align's row spawns a merge conflict session (`internal/loomshed/interruptpolicy.go:41`).
   - A pawl run moved off Parent-Align leaves that session running unguarded. D13 names only PR-Review.

5. **LOW, design** — `lym beam merge-parent` after ready, unguarded by any card.
   - `mergeParent` judges only `judgeLandEntry` (`internal/beamcli/mergeparent.go:29-35`) and never reads the ready record.
   - After `lym beam ready`, it still files a darn member under the beam. Card 7's hold never starts that member, and it blocks the landing until cancelled.
   - Cards 12–14 refuse once ready; this verb recurs unguarded. Card 7's notice does name the cancel.

Tool calls:
1. Grep of `internal/beamcli/mergeparent.go` judgments, `shedengine` CommitStatus call sites, and unknown-row refusals in shedverbs, loomcli, sheddriver and shedengine. My context lacked merge-parent's guards, whether goto and step writes reach CommitStatus, and whether any check before the step refuses a retired row.
