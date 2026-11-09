```yaml
lens: bound-breaker
outcome: reported
```

Lens bound-breaker: I attacked the record's claims and bounds with concrete counterexamples. I break five claims, and the D2 resume-flags claim is weak.

**Findings**

1. **BLOCKING — D5 / D3 (a run already running here goes on, so "adopt never starts work" breaks).**
   - D5 keeps today's rule that a run whose pair `LivePair` reads live here is not paused and reports `running here`.
   - The record is silent on today's step 5, which the record's D3 drops without saying. That step lifts the live owner entry of every slug adopt did not hold, through `tenderengine.Lift` (`internal/adoptcli/doc.go`, lines 168 and 105).
   - `LivePair` reads a pair live under any slug-scope pause, the owner pause included (`doc.go` lines 54-55).
   - Counterexample:
     - Machine B drove run R.
     - Machine A adopted R, so B owner-paused R, and R's strands stayed alive.
     - B now adopts back. R reads `running here` and gets no adopt pause.
     - Step 5, if kept, lifts R's owner entry.
     - Even if step 5 is dropped, B's dobby `liftOwnerPause` lifts `OwnerCaller` entries once the slug is owned here.
     - R goes on with no operator resume.
   - Fix: give a live-here run that carries an owner entry the adopt pause too, or convert its owner entry in place. State what happens to step 5.

2. **BLOCKING — D12 against D11 (the report never lists the old owner's archives).**
   - D12 reads archive tags in the `reconcile` step, step 1.
   - The old owner's dobby archives only once it reads the foreign owner. That needs the owner write, step 4, then a Bolt sync and a probe tick on the old machine.
   - So in the adopt that moved the owner, D12's read always comes before D11's push. D12's stated purpose, "the new owner sees what the old owner had not pushed", is not met.
   - Only a later adopt, or none, would ever list them.
   - Fix: read the archives at a later point that waits for them, or show them in `lym beam status` or another read the operator repeats.

3. **MEDIUM — D4 against D11 (a stale record on the losing machine disables the owner exit and the archive).**
   - D4 counts `interrupted` and `stopped` records as unfinished, with no time limit, and suppresses the dobby's foreign-owner exit.
   - D11's archive and owner pause run on that exit path.
   - Counterexample:
     - B's adopt of root dies after the owner write.
     - The operator goes back to A and runs `lym adopt root` there.
     - B's record still names root. B's dobby and orch watcher never exit, B's orch keeps running under a foreign owner, and B never archives its unpushed tips.
   - The bound ("idle, not running") does not cover this; the orch is not idle.
   - Fix: make the suppression hold only while the subject's owner is not another machine's write made after the record started. Alternatively, run D11 even under suppression once the record is not `running` with a live worker.

4. **MEDIUM — D10 against D2 (an unfinished root adopt blocks `lym s` everywhere).**
   - D2 refuses an adopt whose subjects differ from an unfinished record's. `--here` is such an adopt.
   - An interrupted `lym adopt root` therefore makes `lym adopt --here` refuse in every beam, even beams this machine owns. `lym s` then stops before `beam start`.
   - Two `lym s` calls in different beams at the same moment do the same: the second worker loses `adopt.lock`, and its caller sees a running record with another subject.
   - Fix: let `--here` reach `owned_here` without conflicting with a record of other subjects, or define how the two coexist.

5. **MEDIUM — D11 against `PATTERN-pair-remote` (the archive push takes no teardown lock).**
   - The Slug remote order rule says that touching a slug's pair or origin branches goes under its teardown lock.
   - The existing archive pushes run under that lock (`internal/fabricengine/archive.go` comment: "A teardown archives while it holds the pair's teardown lock exclusively").
   - D11 tags and pushes from the old owner's dobby and names no lock.
   - A teardown or Finalize running on the old machine at the same time can delete the branch between the tip read and the tag push.
   - D11 neither takes the lock nor names the entry as changed.

6. **LOW — D2 (record write order not stated).**
   - D1 and D2 do not say whether the worker writes the record before the `reconcile` step starts.
   - If the reconcile pulls the foreign owner first, the watcher and dobby can act in that gap. This is the original finding.
   - State that the record is written, `running` with step `reconcile`, before any step runs.

7. **LOW — D6 (paused runs only).**
   - "The runs stay paused, so no run goes on under the old lym" does not hold for `running here` runs, which D5 leaves unpaused (see finding 1).

8. **LOW — D11 (task starts not stated).**
   - While a failed archive keeps the old owner's dobby up, D11 says "starting no pass" but says nothing about task starts on the foreign-owned subject.

9. **LOW — D2 (new flags silently ignored).**
   - "Taking the call's flags over the record's" has no effect for `--keep-local` or `--drop-local` once `reconcile` is done.
   - A resume carrying them ignores them without saying so.

**Claims attacked**

| Claim | Result |
|---|---|
| D1: the worker outlives its caller (setsid, runproc `HubOwner`) | held |
| D1: a non-zero exit at the cap stops `lym s` (`runArgvs` stops at the first non-zero exit) | held |
| D2: a resume never repeats a step's effect (steps idempotent) | held |
| D2: the lock-loser race | held |
| D2: flag override on resume | weak (finding 9) |
| D3: order | held |
| D4: suppression bound | broken (findings 3 and 6) |
| D4: done record exits as today | held |
| D5: Held gives no pass, then a full pass at the lift (`planPass` checks `run.Held` first; `trigger.Lift` gives full) | held |
| D5: only a hand resume lifts the pause, and a hibernate wake leaves it alone | held |
| D5: "never starts work" | broken (finding 1) |
| D5: `already_paused` | held |
| D6: owner before deploy keeps runs paused | partly broken (finding 7) |
| D7: dobby without orch starts no adopted work | held, bar the fix task Scope already excludes and finding 1 |
| D8 | held |
| D9 | held |
| D10: a stale Bolt cannot skip the adopt | held |
| D10: no local-commit stop on an owned-here `--here` (`wouldStop` judges only `owned_elsewhere`) | held |
| D10: coexistence with an unfinished record | broken (finding 4) |
| D11: a tag push moves no branch | held for `PATTERN-owner-machine` |
| D11: the lock | broken (finding 5) |
| D11: retry while up | held |
| D12: timing | broken (finding 2) |

**Tool calls**

1. Read `internal/dobbyengine/passes.go` lines 100-160 (`planPass`): my context had only the subagent's summary of how Held and the lift give a full pass.
2. Read `pattern/PATTERN-pair-remote.md`: my context lacked the slug teardown-lock rule that bears on D11's push.
