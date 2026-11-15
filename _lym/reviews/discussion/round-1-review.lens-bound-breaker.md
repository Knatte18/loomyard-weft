```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked each of the record's bounds, guards and claims about existing code.

**Claims attacked**
1. D4, "lym replays as today and reads the run's own record from the result": broken (F1).
2. D4 criterion, "an adopter's takeover push rejected over the old owner's later transition replays and lands": broken (F1).
3. D7/D8, "a hold stands until the adopt that wrote it clears it or `lym adopt --release`": broken when the adopter dies (F2).
4. D7, "under `adopt-fix`, the hold's fix task alone gets `act`", alongside D7's hub pause-record entry: broken (F3).
5. D7 Bound, "until then its step in flight runs to its transition": broken (F4).
6. D3, "a pause or cancel of a run another machine owns is refused … so only the owner writes a run's records": broken by a stale owner read (F5).
7. Goal "stop every run at any moment" against D3's cancel and pause, which push before they act, with D2's offline wait: broken (F6).
8. D1 `cancelled`, "a finished cancel (D3)": broken, since D3 records no finish (F7).
9. D1 act rule against `PATTERN-owner-machine`'s "a landing in flight is never stranded": broken under a hold (F8).
10. D5, "gives each machine one identity": broken for two machines with the same name (F9).
11. D8 step 1, the race between two adopters: held for the same subjects; the re-read clause is narrower than the up-front refusal (F10).
12. D9, a holder lym has not registered is refused: holds as a guard, but it reproduces A6/A7 on the first adopt after the upgrade (F11).
13. D8, "a re-run after an interruption goes on from where the state stands": held, except the try count (F12).
14. D10 lease, "a lost lease … writes nothing": held. A machine that never read the ref leases on "absent" and fails safe.
15. D10 delta upload of the changed last chunk: held. Unchanged chunk blobs are identical objects.
16. D10, "one killed mid-push leaves the ref as it was": held, since a ref update is atomic.
17. D11, "`lym tender resume` pushes no pair": held under D2.
18. D11 orphan condition (no entry and no batten record, no commit, no live process): held.
19. D3, "an absent cache reads as empty and refuses nothing": held. Pause intents are rewritten at catch-up.
20. D2, "the next producer starts only once the push has landed": held.
21. D9 runproc registration "pruned when the process dies": held, since runproc keys on pid and start time.

**Findings**

- **F1. BLOCKING, design. D4 (hand-over path) and its criterion "an adopter's takeover push rejected … replays and lands".**
  Summary: the takeover is detected by replaying local commits and reading the result, but the takeover entry and the old owner's transition both edit the same run record, so the replay stops on a conflict and produces no result to read.
  Evidence: `internal/fabricengine/replay.go:45-48`. `ErrReplayConflict`: "a records replay stopped on a commit it could not apply … the starting tip is restored". Its comment at :674 says only "a file only one side changed never conflicts".
  Counterexample:
  - A's transition commit and B's takeover entry both change the batten record (the prime's `status.json`), and the child `status.json`, for the slug.
  - A's push is rejected and its replay conflicts, so A never reads the takeover and never hands over.
  - D2 names no wait for a non-transient failure, so A is left with a failed-push record and its processes still running.
  - The adopter's own replay in D4's last bullet hits the same conflict, so "replays and lands" fails too.
  - Fix: read the fetched origin record for a takeover before any replay, or keep the takeover entry in a file no transition writes.

- **F2. BLOCKING, design. D7 Bound and D8 (`--release`, step 1).**
  Summary: a hold whose adopter died or stays off stops the whole hub on every machine, and no stated path clears it.
  Evidence: D7 Bound, "A hold stands until the adopt that wrote it clears it or `lym adopt --release`". D8 step 1, "A hold another machine wrote refuses the adopt". D8, "`lym adopt --release` clears a hold over a failing check".
  Counterexample:
  - Machine A runs `lym adopt` and is switched off after step 1.
  - Machine B's dobby reads the hold, and its act rule answers `wait` for every run.
  - `lym adopt` on B is refused, naming A.
  - `--release` is defined only "over a failing check", and nothing says it may clear an `adopt` hold, or a hold another machine wrote.
  - The hub cannot go on on another machine, against the Goal's "go on later … on another one after it pulls".

- **F3. BLOCKING, consistency. D7 bullets 2 and 3 against D8 step 8.**
  Summary: the hub pause-record entry a machine writes on reading a hold holds the fix task's own processes, so the one run the act rule lets act cannot run.
  Evidence: `internal/pauserecord/doc.go`: "A hub entry covers every slug". D7: "A machine that reads a hold writes a hub entry in its own pause record, so the network gate and the turn interruption hold its runs' processes"; "Under `adopt-fix`, the hold's fix task alone gets `act`".
  Counterexample:
  - On the adopter, the `adopt-fix` hold yields a hub entry.
  - The fix task's agent turns and network calls wait in `WaitUncovered`.
  - The fix task never lands, and the hold never clears.
  - The same entry holds the children that steps 5 and 7 spawn: the deploy command and the gate run. `SetGateExemptAll` "never reaches a child", per the same doc.

- **F4. LOW, consistency. D7 Bound.**
  Summary: an old owner that has not yet pulled Bolt runs more than "its step in flight". It starts new producers and pushes transitions until its next Bolt pull, up to the dobby's pull interval.
  Evidence: D7 Bound; state map 4.3, "pulls Bolt on a probe tick, at most every 5 minutes (`dobbyengine/handover.go:17-32`)".
  The D4 Bound covers the outcome. D7's wording understates the window.

- **F5. MEDIUM, design. D3, the owner-only refusal.**
  Summary: the refusal does not say which Bolt it reads the owner from. A stale local Bolt lets a former owner write a pause or cancel intent into a run it no longer owns, and the guard's bound is unstated.
  Evidence: D3, "A pause or cancel of a run another machine owns is refused, naming that machine"; `PATTERN-owner-machine`, "A resume with origin unreachable reads the local Bolt".
  Counterexample:
  - B adopted the run, and A has not pulled Bolt.
  - `lym tender pause &lt;slug&gt;` on A passes the guard and pushes an intent.
  - B's next push is rejected, replays (D4) and picks up the intent.
  - B's act rule reads "this machine's pause record coverage", which only catch-up or tender writes, so B answers `act` while D1 reads `paused`.

- **F6. MEDIUM, design. D3 against D2's Bound and the Goal.**
  Summary: with origin unreachable, the operator cannot cancel a run, and the slug pause path is undefined, because each verb pushes before it acts. No Decision states what those verbs do offline.
  Evidence: D3, "`lym batten cancel` commits and pushes it before it ends anything"; "`lym tender pause &lt;slug&gt;` … committing and pushing before it returns"; D2 Bound, "no run acts while origin is unreachable".
  - An agent step in flight goes on offline until its transition.
  - Only the hub-wide machine-local pause (Scope Out) stops it, and no Decision names that as the offline stop.

- **F7. MEDIUM, design. D1 states 2 and 3 against D3.**
  Summary: `cancelled` reads "a finished cancel", but D3's cancel intent records only when it started, the machine and remove or keep. Nothing tracked marks the finish, so `cancelled` and `cancelling` cannot be told apart from tracked records.
  Evidence: D1 list items 2 and 3; D3 first bullet.

- **F8. MEDIUM, consistency. D1 act rule and D7, against `PATTERN-owner-machine`.**
  Summary: the carve-out for a Finalize past its parent-side merge exempts it from `hand_over` only. A hold makes it `wait` "for every run on every machine", so a landing half done is stranded for as long as the hold stands. At the fix-try limit, "the hold stays".
  Evidence: D1, "`hand_over` when another machine owns the run, unless the run is `landing` past its parent-side merge" and "`wait` when a hold covers it"; D7, "the act rule answers `wait` for every run on every machine"; `pattern/PATTERN-owner-machine.md:23-24`, "A landing in flight is therefore never stranded".

- **F9. LOW, design. D5.**
  Summary: two machines that resolve to the same current name register as one machine. Examples are the same short hostname with no `machine.json`, or the same chosen name. The registry refuses only a clash with another machine's former name, so both read each other's runs as owned here.
  Evidence: D5 bullet 2, "A registration whose former name is another registered machine's current name is refused"; state map 3.3, "without it the name is the lowercased short hostname".

- **F10. LOW, consistency. D8 step 1.**
  Summary: the rejected-push re-read refuses only "an owner or hold of these subjects". Holds are hub-wide and carry no subjects, so two machines adopting different beams at once can both push a hold, against the up-front rule "A hold another machine wrote refuses the adopt".
  Evidence: D8 step 1; D7, "A hold covers the whole hub".

- **F11. LOW, design. D9 at the upgrade.**
  Summary: an orch Claude process launched by the pre-upgrade lym is not in runproc. The first orch start after the upgrade then refuses over it, naming its pid, which is the A6/A7 holder once more, and no bound or migration is stated.
  Evidence: D9 bullets 2 and 4; bound-breaker lens finding 4 of the Discussion: "The orch packages register no session".

- **F12. LOW, design. D8 step 8.**
  Summary: the try count that bounds "within `adopt_fix_tries`" lives in the adopt subject directory under Bolt's `.lym`. D3 classes that directory as cache, which "reads as empty and refuses nothing", so a lost or rebuilt cache resets the limit.
  Evidence: `internal/adoptcli/fixloop.go:213` (`readTries(deps.Dir)`) and `:386` (`subjectDir`); state map 3.2, "`adopt/…/&lt;subject&gt;/{check.json,fix-tries.json,...}`"; D3's cache list ("adopt check views").

Tool calls: none.
