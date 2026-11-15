```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens. I checked pairs of decisions that act on the same state and ran each one's paths with the other in force, then checked the acceptance criteria and constraints against the decisions they touch.

**Pairs checked**

1. D1 act rule × D7 hold: broken. The `landing`-past-merge exemption covers `hand_over` only, so such a landing waits under a hold.
2. D7 hold's pause record entry × D7/D8 `adopt-fix` fix task: broken. The hub entry holds the processes of the fix task the act rule says may act.
3. D1 `cancelled`/`cancelling` × D3 cancel intent: broken. D3 records no mark for a finished cancel.
4. D1 act rule pause input × D3 pause intent: broken. The record leaves unclear whether a `paused` state answers `wait`.
5. D2 rejected-push replay × D4 takeover entry: broken. The takeover edits the same record the old owner's transition edits, so the replay stops on a conflict.
6. D4 × declared `PATTERN-run-fsm` text: broken. D4 says "replays as today"; the declared text says "never replays".
7. D3 pause and cancel verbs × D2 offline wait: broken. Neither says what a slug pause or a cancel does with origin unreachable.
8. D7 hold lifetime × D8 step 1 refusal and `--release` scope: broken. A dead adopter's `adopt` hold has no clearing path.
9. D8 step 6 transcript restore × D10 mirror × D7 hold delay: broken. The old owner's mirror lands after the adopter's restore, and the adopter's lease is lost from then on.
10. D7 upgrade conversion × D3 owner-only intents: held. The conversion applies only to runs this machine owns.
11. D5 registry write × D7 hold write × D8 step 1 single commit: held.
12. D6 step 3 owed push × D4 takeover: held. A rejection routes to D4.
13. D6 steps 4–5 × D7 hold pause entries: held.
14. D9 ends × D10 mirror at stop: held as stated. Handover's loss of up to one mirror interval is unstated (folded into finding 6).
15. D11 kept pair × D3 cancel intent's keep: held.
16. D11 orphan rule × D6 step 4 local removal: held.
17. D2 no deferred push × D11 `lym tender resume`: held.
18. D8 step 8 × D1 `queued` × D7 "fix task alone acts": held at the act-rule level, broken at the process level (see finding 2).
19. Acceptance criterion D7 "under `adopt-fix` the fix task alone gets passes" × D7's hub pause entry: broken (finding 2).
20. Constraint `PATTERN-refusal-way-forward` × D7/D8 dead-adopter hold: the refusals have no way forward an agent may be told (finding 3).

**Findings**

1. **BLOCKING · design · D2 × D4: a rejected push replays before it reads the takeover, and that replay conflicts.**
   - D4 has the old owner "replay as today and read the run's own record from the result".
   - The takeover entry is written into the batten record and the child record. The old owner's transition commits edit those same records.
   - Replaying its commit over the takeover therefore stops with `ErrReplayConflict`, which restores the starting tip. There is no result to read the takeover from.
   - The old owner never hands over, so D4's first acceptance criterion cannot pass.
   - The fix is to read the takeover from origin's fetched record before any replay.
   - Evidence: `internal/fabricengine/replay.go:45-48`; `decision-record.md:118-120`.

2. **BLOCKING · consistency · D7 × D8: the hold's hub pause entry holds the fix task that `adopt-fix` lets act.**
   - D7 has every machine that reads a hold write a hub entry in its own pause record, "so the network gate and the turn interruption hold its runs' processes". That includes the adopter while its `adopt-fix` hold stands.
   - A hub entry covers every slug, so the fix task's own processes are held even though the act rule answers `act` for it.
   - Step 7's check also runs while the `adopt` hold stands. Its `lym gate test` children install their own gate, which the hub entry holds.
   - So the fix loop and D7's criterion "under `adopt-fix` the fix task alone gets passes" cannot both hold.
   - Evidence: `internal/pauserecord/doc.go` ("A hub entry covers every slug"; "The exemption never reaches a child"); `decision-record.md:181-182, 216-218, 360`.

3. **BLOCKING · design · D7 × D8: a hold whose adopter died has no way out.**
   - D7: a hold stands until the adopt that wrote it clears it, or until `--release` does.
   - D8 step 1: another machine's hold refuses the adopt.
   - `--release` is stated only for "a hold over a failing check", and nothing says one machine may release a hold another machine wrote.
   - So an adopter that dies after step 1, or an `adopt-fix` hold whose machine stays off, holds every run on every machine with no stated path out.
   - That contradicts the Goal's "go on later … on another one". No bound for a hold with no live owner is stated.
   - Evidence: `decision-record.md:9, 195, 206-207, 223`.

4. **MEDIUM · design · D10 × D8 step 6 × D7 bound: the old owner overwrites the restored mirror and the adopter loses its lease.**
   - The old owner learns of the takeover only at its next Bolt pull, up to the dobby's pull interval later. Until then its act rule does not answer `hand_over`, so its orch keeps mirroring at turn end.
   - Its lease is on its own last-written value, so the push lands after the adopter's step 6 restore.
   - The adopter's later mirrors hold a lease on the value it read at restore, so each one loses the lease and "writes nothing". Its mirror stays stale until it re-reads, which no decision states.
   - A later restore then gets the old owner's diverged transcript.
   - Evidence: `decision-record.md:193-194, 215, 263-269`.

5. **MEDIUM · consistency · D1 × D7 × D6/D9: the `landing` exemption stops at a hold.**
   - D1 exempts a `landing` past its parent-side merge only from `hand_over`. Its `wait` rule then applies to "every run on every machine" under a hold.
   - So a Finalize past its merge freezes while the hold stands, and indefinitely when an `adopt-fix` hold stays at the try limit.
   - D6 and D9 say it "finishes here", and `PATTERN-owner-machine` says a landing in flight is never stranded.
   - The record should state which rule wins.
   - Evidence: `decision-record.md:51-52, 180, 221, 242`; `pattern/PATTERN-owner-machine.md:23-24`.

6. **MEDIUM · consistency · D1 × D3: `cancelled` cannot be told from `cancelling`.**
   - D1 reads `cancelled` from "a finished cancel (D3)" and `cancelling` from an intent whose cancel has not finished.
   - D3 records only the intent: when it started, the machine, and remove or keep. It names no finish mark.
   - Without one, D1's order cannot separate the two states, and D11's teardown condition rests on `cancelled`.
   - Evidence: `decision-record.md:36-37, 88-90, 289`.

7. **MEDIUM · design · D3 × D2: a slug pause or a cancel with origin unreachable is unstated.**
   - D3's `lym batten cancel` commits and pushes "before it ends anything", and `lym tender pause &lt;slug&gt;` and `resume` push "before it returns".
   - D2 makes an unreachable origin a wait.
   - No decision says whether these verbs refuse, block or record locally offline. So whether the operator can cancel a run "at any moment" (the Goal) is open, and the bound of each verb's offline path is missing.
   - Evidence: `decision-record.md:8, 90-92, 69-70, 77`.

8. **LOW · consistency · D1 × D3: the act rule's pause input does not cover a pause intent.**
   - The act rule's pause input is "this machine's pause record coverage". D3 writes that entry only at the pausing verb and at a catch-up.
   - A pause intent reaching the owner any other way gives FSM state `paused` but no local coverage. One example is a non-owner's write admitted by a stale owner read, then pulled through a replay. In that case D1 does not say that `paused` answers `wait`.
   - Evidence: `decision-record.md:38, 49, 52, 92-98`.

9. **LOW · consistency · D4 × declared `PATTERN-run-fsm`: "replays" against "never replays".**
   - The declared text says a run whose record holds another machine's takeover "hands over, never replays". D4 replays first, then reads.
   - One of the two texts should change. Finding 1's fix aligns them.
   - Evidence: `decision-record.md:119, 382`.

Tool calls: none; every item above was already in my inherited context.
