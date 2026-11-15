```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each Decision's mechanism end to end on one concrete case and checked the end state against the Decision's stated purpose.

**Walkthrough list**

- D1, old owner A whose run sits at `Finalize` past its parent-side merge while B's adopt hold stands: **purpose not met.** The carve-out only skips `hand_over`, and the hold's `wait` still applies, so the landing is stranded. A run cancelled with keep never reaches the `cancelled` state either (see D3).
- D2, a task-pair transition whose push is first rejected over another run's `main-weft` commit, then made with origin unreachable: **met.** It replays and pushes, then waits with the push owed and retries.
- D3, `lym batten cancel &lt;slug&gt;` with keep, with origin up and then with origin down: **not met.** The cancel removes the batten record the intent lives in. Offline, the cancel can end nothing.
- D3/D7, the upgrade conversion of an `adopt` entry on an unstarted released task: **not met.** There is no batten record to hold the intent.
- D4, A pushes a transition after B's takeover entry has landed: **not met.** A's replay conflicts on the same status file, so the hand-over never triggers.
- D5, an owner recorded as `osl-1046` and read on `neve` after `osl46` registers: **met.** Before the registration it still reads as two machines, which D5 accepts.
- D6, a laptop wakes from sleep and finds one owned run with owed commits and one run another machine took: **met.**
- D7, B's `adopt-fix` hold with the fix task running on B: **not met.** B's own hub pause entry holds the fix task's processes. The cancel way forward is also refused.
- D8, B adopts and its check fails, then B goes off for good and the operator runs `lym adopt` on A: **not met.** A is refused, and `--release` is not stated to clear another machine's hold.
- D9, an orch launched after the upgrade, its strand removed and its Claude process still alive, then an orch start: **met.**
- D10, the orch moves from A to B with a mirror older than 10 minutes, then is restored on B: **met** within the stated bound.
- D11, the kept `claude-account-weights` pair: **not met.** No batten record or cancel intent survives the cancel, so nothing lists it, and it reads as an orphan once its entry goes.

**Findings**

1. **BLOCKING · design · D4 hand-over trigger.** The takeover entry and the old owner's transition edit the same batten and child status files. A's replay therefore stops on `ErrReplayConflict` with its starting tip restored. It yields no "result" to read the takeover from, so A never hands over, never ends its processes and never archives. D4's first acceptance criterion cannot be met as written. Fix: read the takeover from the fetched origin record before any replay, or keep the takeover in a file no transition writes. Evidence: `internal/fabricengine/replay.go:45-48`, `:425`; D4 bullets 1–2.

2. **BLOCKING · design · D7 and D8 under `adopt-fix`.** A machine that reads a hold writes a hub entry in its own pause record. That includes the adopter, and "a hub entry covers every slug", so the network gate and the turn interruption hold the fix task's run processes. The act rule answers `act` for the fix task, but its processes wait at the gate and the fix loop can never land. D7 states no exemption for the fix task's slug. Evidence: `internal/pauserecord/doc.go` ("A hub entry covers every slug", `WaitUncovered`); D7 bullets 2–3.

3. **MEDIUM · design · D7 and D8, the fix loop's way forward.** `lym batten cancel` refuses "while a hub pause covers the prime". Under a hold, every machine's pause record carries the hub entry D7 writes. So the way forward the fix loop prints for a halted fix task (`lym batten cancel &lt;fix&gt; --remove`) is refused for as long as the hold stands. Evidence: `internal/battencli/cancelcmd.go:62-65`; `internal/adoptcli/fixloop.go:290`.

4. **MEDIUM · design · D3, D1 `cancelled` and D11 kept pairs.** Today a cancel "removes the run's records and scratch and commits that removal". D3 puts the cancel intent, including keep or remove, in that batten record but never says the cancel stops removing it or records its finish. This has three effects:
   - `cancelled` (D1 state 2) has no record to read.
   - D11's "a kept pair's batten record keeps it out of the orphans" has no record to rest on, so a kept pair whose entry is removed is torn down after the grace, against the keep.
   - The motivating `claude-account-weights` pair, cancelled before the upgrade, reads as neither a kept pair nor an orphan while its entry stands, so nothing reports it.
   
   Evidence: `internal/battencli/cancelcmd.go:54-58`; D3 bullet 1; D11 bullets 2–3 and its Bound.

5. **MEDIUM · design · D7 and D8, a hold no live process owns.** Take B adopting: B writes an `adopt` hold (step 1) or an `adopt-fix` hold (step 8), then goes off for good. A's `lym adopt` is refused naming B. D8's `--release` is described only as clearing "a hold over a failing check", and no Decision says it may clear a hold another machine wrote. The hub then stays held on every machine, against the Goal's "go on later … on another one after it pulls". Evidence: D8 step 1 and the `--release` paragraph; D7 Bound.

6. **MEDIUM · consistency · D1 against D7 and D9, a landing past its parent-side merge.** D1 ranks `hand_over`, with its carve-out, before `wait`. D7 says a hold makes every run on every machine `wait`. A Finalize past its parent-side merge on the old owner therefore keeps its processes (D9) but is frozen by the hub entry. If the hold stays at the try limit, it stays half-landed indefinitely. This contradicts the owner-machine background's "a landing in flight is never stranded". Evidence: D1 act-rule bullets; D7 bullet 2; `pattern/PATTERN-owner-machine.md:23-24`.

7. **MEDIUM · design · D3 offline.** Today an offline cancel "only ends the run's processes and strands, leaves its marker". D3 requires the cancel to commit and push the intent "before it ends anything", and it retires the marker. Offline, the cancel can then end nothing, and no Decision states what the slug-scoped cancel, pause and resume do without origin. Evidence: `internal/battencli/cancelcmd.go:67-68`; D3 bullets 1–2.

8. **LOW · design · D7 upgrade conversion and D3.** Today's adopt pause also covers each unstarted released task (no batten record). D7 turns such an entry into "a run pause intent (D3)", but D3 keeps that intent in the batten record. Creating one makes the task read as not `queued` (D1 state 9 needs no batten record) and as no other state, since it has no row. Evidence: `pattern/PATTERN-dobby-runs.md:21`; D1 states 8–9; D7 bullet 5.

Tool calls:
1. `grep` of `internal/battencli/cancelcmd.go` for status, draft and marker writes, because my context did not show what a cancel does to the batten record.
2. `sed -n 50,80p internal/battencli/cancelcmd.go`, the cancel verb's help text, which states the record removal, the offline behaviour and the hub-pause refusal.
