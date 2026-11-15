```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran one concrete case through each of D1–D11. I checked each step's input against what the step before it produces, and the end state against the decision's purpose.

**Findings**

1. **BLOCKING — D8 step 2 depends on step 3.**
   - Summary: the takeover commit in step 2 needs local pair branches that only step 3's catch-up creates.
   - Case: osl adopts root from neve, and osl has no pair for moved run R.
   - Step 2 must commit a takeover onto R's pair records branch, and R's batten record if it has none here, then push.
   - But D8 step 3 is what "adds the pair of each moved run that has none here, from origin". So step 2 has no branch or worktree to commit on, for every run of a fresh move.
   - Fix: run the catch-up and pair add before the takeover, or write the takeover against the fetched origin tip.

2. **BLOCKING — D4's rule, combined with D8 and D6, turns on the adopter.**
   - Summary: D4 fires on "a commit naming another machine", so the adopter stops the very runs it is taking over.
   - Case: osl writes the owner in step 1. Before osl's step 2, neve, which has not noticed yet, pushes one more transition of R.
   - osl's takeover push is then rejected. osl fetches and finds a commit naming neve, so D4 tells osl to stop acting on R, end its processes, archive and reset. The takeover never lands.
   - Same case in D6 step 3: neve adopts R back, its local branch still holds old neve commits, and origin holds osl's commits. That is "origin moved by another machine", so D4 runs on the new owner.
   - Fix: D4 must also read the Bolt owner. Stop only when the owner is another machine. When the owner is here, rebase the takeover or reset to origin and go on.

3. **BLOCKING — D4 judges by branch, but a records branch holds many runs.**
   - Summary: one shared branch makes a beam adopt stop the root's own runs on the old owner.
   - Root runs' batten records all live on `main-weft`, and a beam's pawl batten record may sit in its parent's weft.
   - Case: osl adopts beam X while neve keeps root. osl's takeover commit for X's pawl record lands on `main-weft`, a branch osl does not own.
   - neve's next push of an unrelated root run is then rejected over a commit naming osl. D4 stops that root run and resets `main-weft` to origin, though neve still owns it.
   - Fix: the rule must be per run, judged by the run's owner, not by who committed on the branch.

4. **MEDIUM — D7: one `hold` field cannot hold two holds.**
   - Summary: a later hold overwrites an earlier one.
   - Case: neve adopts beam X (`adopt` hold over X) while osl adopts beam Y. osl's step 1 writes `hold` over Y, and the refusal only covers "these subjects".
   - The single field now names Y, so X's runs are unheld mid-adopt. The same happens with an operator `pause` hold and an adopt at once.
   - Fix: make `hold` a list keyed by subject or kind.

5. **MEDIUM — D7 and D3: holds no longer stop running processes.**
   - Summary: a pause stops new actions but not a run already moving.
   - Today a pause holds a run's own processes through the network gate, which reads the local pause record (commit `dce431f4`).
   - D7 leaves only hibernate's entries and the interrupted sessions in that record. D1's act rule is asked only by pass, run, resume and goto, not at each transition.
   - Case: `lym tender pause` hub-wide (or a run-scope pause) while R's step loop runs. R keeps stepping, and D2 keeps pushing its transitions until the pass ends.
   - Fix: say that the gate, or each transition, reads the tracked hold and pause intent.

6. **MEDIUM — D8 holds the subjects, not the hub.**
   - Summary: the hold falls short of the operator's rule.
   - The board's operator decision says "Nothing else happens in the hub until adopt is done".
   - D8 step 1 holds only the subjects. Case: `lym adopt X` (a beam) leaves root's dobby passing and claiming its own tasks during the adopt and the fix phase.
   - Fix: either hold hub-wide or record why subjects alone are enough.

7. **MEDIUM — D9 misses the recorded A6/A7 case.**
   - Summary: the stale holder will not pass D9's ownership test, so the fix lands in the refuse branch.
   - In A6/A7 the old orch's strand was already removed by the watcher's `RemoveStrand`, and its Claude process survived (exploration finding).
   - D9 ends a holder only if it is the pane process of a live orch strand, or a process `internal/runproc` registers for the worktree's orch.
   - `runproc` has no orch role, only `RoleSession` for pane processes. Nothing says the orch's Claude process is registered, or that the record survives `RemoveStrand`.
   - So the stale holder reads "not owned", D9 refuses, and the operator must exit it by hand.
   - Fix: say how the orch's Claude process is registered, and that the registration outlives strand removal. Or identify it by session id and cwd.

8. **MEDIUM — D10's length test can resume a stale fork.**
   - Summary: "origin holds more of the transcript" is the wrong comparison when two machines diverged.
   - Case: neve's orch ran 8 minutes past its last mirror and then stopped. osl restored origin's mirror, resumed and added 3 turns.
   - When neve adopts back, its local copy can be longer than origin's, though origin's holds turns neve lacks.
   - neve then resumes its stale fork, and its next mirror replaces the ref, losing osl's turns for good.
   - Fix: compare by the mirror's writer machine or its last line, and let origin win when another machine wrote it after this machine's last mirror.

9. **MEDIUM — D6 step 4: a full teardown would delete origin's branch.**
   - Summary: the catch-up must remove only the local copy of a pair another machine still drives.
   - `PATTERN-pair-remote` says a teardown archives and deletes the weft branch and settles the board claim.
   - Case: a catch-up on neve finds R owned by osl and removes R's pair "through `internal/pairteardown`". As stated, that can delete R's origin weft branch while osl drives R, and touch the claim.
   - A local-only removal exists (fabric remove under an `owned_elsewhere` verdict), but D6 does not name it.
   - Fix: D6 must name the local-only removal.

10. **MEDIUM — D5 and D4: old names and old commits.**
    - Summary: names chosen before the registry, and commits made before the trailer, are left out.
    - Case: osl was renamed to `osl46` before the registry existed. Its first owner write registers `osl46` only. D5 does not say it copies `machine.json`'s `names[]`, so neve still reads `osl-1046` as another machine. That is A8 unfixed.
    - D4 also says nothing about a commit with no `Lym-Machine` trailer, which is every commit before the upgrade. "Names another machine" is undefined for it.

11. **LOW — D1: overlapping states.**
    - A child `failed` at the Finalize row is both `halted` and `landing`.
    - A `cancelling` run whose child is `running` is two states.
    - The state list sets no order between them.

12. **LOW — D11: the grace can be zero.**
    - A bench pair idle for days gets its "says when it goes" notice at first sight and is torn down on the same tick.
    - The grace runs from the last commit, not from the notice.

**Decisions walked**
- D1 — run at Run-Shed whose pair is absent on the reading machine: purpose missed. The child record lives on the pair's weft branch, and D6 removes that pair for a run owned elsewhere. The record does not say the record is read from the fetched origin branch.
- D2 — loom transition with origin down: met. The run waits with the push owed.
- D3 — cancel on neve interrupted, read on osl: met once the intent is pushed. The pause intent fails through finding 5.
- D4 — owner race and shared branch: missed (findings 2 and 3).
- D5 — rename `osl-1046` to `osl46` before the registry existed: missed (finding 10).
- D6 — cold dobby start, run now owned elsewhere: met except the removal mode (finding 9).
- D7 — hub pause during a running step: missed (findings 4 and 5).
- D8 — osl adopts root with no local pairs: missed (findings 1, 2 and 6). The fix-task loop itself is met.
- D9 — A7 case: missed (finding 7).
- D10 — move to osl and back to neve: missed on divergence (finding 8). A one-way move is met.
- D11 — cancel without keep, bench pair, `rerun` pair: met, bar finding 12.

**Tool calls**
- Read `internal/runproc/record.go`'s role list and grepped orchcli/orchengine for `RoleOrch` and `runproc.Register`: my context had no evidence on whether the orch's Claude process is registered, which D9's holder test rests on.
