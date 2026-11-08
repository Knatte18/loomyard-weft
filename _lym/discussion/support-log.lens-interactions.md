```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of decisions that act on the same state, then each criterion and constraint against the decisions it touches.

**Findings**

1. **BLOCKING** — D7 step 4 with D4/D5: a judged test can count as its own cover.
   - Step 4 counts as a cover "a map test that still exists with the same function hash", and it never excludes the judged test itself.
   - D5 lets `lym gate testmap` run "in the worktree it is called in". An agent or the operator running it in the run's own worktree writes the hub-shared map (D4) from a tree that holds the run's new tests.
   - On the next gate, each new test finds itself in the map with the same hash, so every new test is reported contained, covered by itself.
   - Fix: exclude the judged test's own name from its covers.

2. **MEDIUM** — D5 against D6/D4: the record gives one writer of the map and then lets every worktree write it.
   - D6 says "No other dobby refreshes maps", and the support log rejects "Rebuild in every dobby: beam dobbies would overwrite maps with their branch's tests".
   - D5 lets anyone run the verb from any worktree (beam, run, darn) into the same map path.
   - That reopens what the rejection names: the map holds a branch's tests and file hashes, and the prime dobby's next staleness check compares against them.
   - Fix: bound D5 to the prime parent's worktree, or refuse it elsewhere with a way forward per `PATTERN-refusal-way-forward`.

3. **MEDIUM** — D7 with D8: the step records HEAD, but nothing says the tree it judged is HEAD's.
   - D7 runs "against HEAD" and records HEAD's commit. D8 shows the report only when that commit equals the reviewed HEAD.
   - If a key's gate runs on a dirty tree before the writer or fixer commits, as the darn writer and the Code-Burler-Fix fixer may, the recorded HEAD is the old commit. D8 then reports "no current report" on every round, so the feature does nothing on those keys.
   - The reverse can also happen: the report holds HEAD while judging uncommitted content.
   - The record needs to state what the step judges: the committed tree, or the tree's content, recorded and compared by a tree or content fingerprint.

4. **MEDIUM** — D6 with the board-tidy counter it reuses.
   - `countLandings` in `internal/dobbyengine/tidy.go` counts nothing while a board-tidy agent is in flight, and `LandedSinceTidy` is cleared only when a tidy settles.
   - If D6 triggers on that list being non-empty, it fires every tick until tidy clears it. If it triggers on "a tick that counted", refreshes stall for as long as a tidy agent runs.
   - With `board_tidy_every: 0`, `LandedSinceTidy` is never consumed.
   - D6 needs its own seen-mark over the counted landings, independent of tidy's list and of tidy being in flight.

5. **MEDIUM** — D5's dirty-package skip with D6's trigger.
   - A package D5 skips because it is dirty is retried only when the next landing is counted, not on a "later tick" as the Auto-mode assumptions say.
   - In a prime worktree the operator edits, the map can stay stale indefinitely.
   - Either retry on ticks while stale packages remain, or correct the assumption.

6. **MEDIUM** — D7 step 4 with the tool's removable-set rule (D1/D2).
   - Two new tests in one diff that cover each other are both reported contained, each naming the other.
   - D8 has the reviewer file fold-or-delete findings per test. Nothing carries the `PATTERN-test-speed` rule that only a removable set may go together.
   - Deleting both loses the coverage.
   - D7 should run the greedy pass over the judged tests, or mark mutual covers as "drop one".

7. **LOW** — D7 with gate control flow.
   - The record does not say whether the step runs when a tier fails, or when tiers are carried over from the green record (`internal/gatecli/green.go`).
   - On a carried-over gate the report is not rewritten, so D8 shows "no current report" when it could have been current.
   - State the step's behaviour on a red gate and on a carry-over.

8. **LOW** — the D1 acceptance criterion is ambiguous.
   - "same contained set for the same package and profiles" cannot hold literally: the tool (D2) judges among all eligible tests at one commit, while the step (D7) uses map covers filtered by function hash.
   - Rephrase it as: the same profiles fed to the engine's classifier give the same verdict whichever caller feeds them.

**Pairs checked**
- D1–D2 (engine eligibility, module-wide versus package view): held.
- D1–D7 (eligibility plus the keep and tier filters): held.
- D2–D3 (positions within one run): held.
- D3–D4 and D3–D7 (content keys across commits): held.
- D4–D5 (staleness by file hash, HEAD as the map's commit): held.
- D4–D7 (atomic replace while the step reads): held.
- D5–D6 (in-flight limit, per-package lock): held.
- D5–D6 (dirty skip and retry): broken, finding 5.
- D5–D4/D6 (who writes the shared map): broken, finding 2.
- D5–D7 (map built from a run tree): broken, finding 1.
- D6 with the tidy landing counter: broken, finding 4.
- D7–D8 (report commit against reviewed HEAD): broken, finding 3.
- D7 with the green carry-over and a red gate: underspecified, finding 7.
- D7–D2 (mutual covers against the removable-set rule): broken, finding 6.
- D8–D9 (rubric versus background wording): held.
- The constraints (`PATTERN-gate-slot`, `PATTERN-tolerant-records`, `PATTERN-child-processes`, `PATTERN-config-layers`, no test launching a test binary) against D4 to D7: held. D7 does not itself say it holds a slot, but the Constraints line covers it.
- Acceptance criteria D2 to D8: each matches its decision. The D1 criterion is finding 8.

**Tool calls**
- Read `internal/dobbyengine/tidy.go` lines 100–174: my context had only grep lines for `countLandings` and `tidyStep`, not how the counter behaves while a tidy is in flight or when tidy is off.
