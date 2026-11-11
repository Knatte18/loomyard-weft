```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism end to end on one concrete case and checked the end state against the decision's stated purpose. I made no extra tool calls.

**Findings**

1. **BLOCKING — D7 and D8 together: the audit can never run.**
   - D7 runs the audit "when its queue is empty and it holds a slot".
   - D8 says the upkeep process "ends when its queue is empty", and the dobby starts one only when HEAD moved or the exit record names queued work.
   - D9 takes a slot only for a package job.
   - Case: the plan at H finds every map current, so the queue is empty. The process ends without taking a slot, and the dobby starts no other.
   - So the audit never executes, and D7's replacement for the age limit is dead. The self-check runs only on builds that happen anyway.
   - Purpose not met.

2. **MEDIUM — D1 and D6: re-stamping can make every map a selector miss.**
   - `coverage-test-select`'s D5 treats a map as a miss when its format marker's `Commit` and `BuiltAt` differ from the map's own.
   - D1's re-stamp says "`Commit` becomes `H`" and says nothing about the marker. D10's per-round writes at H have the same gap.
   - Case: map at C, then a body-only landing, then a re-stamp to H. Marker.Commit stays C while Map.Commit is H, so the gate misses on the marker rule.
   - What re-stamping produces is not what the selector accepts. The record must say every upkeep write sets the marker's `Commit` and `BuiltAt` together with the map's.

3. **MEDIUM — D4, D1 and D8: spawn-marked tests can loop forever.**
   - D1 queues "every declared mapped test with no entry".
   - D4 records spawn-marked tests "with that verdict" and never runs them, and it does not say whether a verdict counts as an entry.
   - Case: a package holds an out-of-process test. It is queued and never run, so the queue never empties.
   - D8 then restarts upkeep on every tick ("that record names queued work"). D8's own guard covers only dirty-skipped packages.
   - The record must say spawn-marked tests, and failure-at-H packages (finding 4), are not queued work.

4. **MEDIUM — D5 against D1 and D8: a failed build is re-queued at the same HEAD.**
   - Case: B's map misses at H and the whole rebuild fails. D5 keeps the map at C and records the failure with H.
   - The next plan at the same H, for example after a yield or a restart, still sees C→H as a miss and queues B again. The exit record then names queued work, and the dobby restarts upkeep.
   - D5's "not retried at the same H" is not wired into D1's plan or D8's queued count, so it gives a restart loop or a retry.

5. **MEDIUM — D13: the carrier window does not exist for these tasks.**
   - D13 says changes go on "before that entry's run reaches Plan", through `lym loom decision add`.
   - `testmap-cross-package`, `coverage-test-select` and `gate-scope-coverage` all use recipe `darn`. Darn-Write writes code directly: T4 is at Darn-Write now, and peers show `plan:false`.
   - A discussed darn run reaches no Plan, so the changes must land before Darn-Write starts its writer. For T4 that moment has already passed.

6. **MEDIUM — D9 and D13: the E4 sweep can judge a map that was never rebuilt.**
   - D13 has `redundancy-sweep-existing` run a hand call that "rebuilds it whole" before judging.
   - Today a call that finds the per-package build lock held skips the package (subagent report, item 7). D9 keeps that lock behaviour.
   - Case: the background job holds B's lock, so the sweep's hand call skips B silently, and the sweep judges a partial map.
   - The hand call must wait for the lock or report the skip.

7. **LOW — D13 against D10 and D11: the build order contradicts itself.**
   - D13 orders D10 after the selector, but D10 (side-by-side runs, per-round writes) calls no selector.
   - D13 lets D11 land without D10, yet D11's rationale depends on D10's per-round writes. Without them, ending upkeep loses a whole package's runs.

8. **LOW — D1: a plan rule sits outside the three outcomes.**
   - D4 drops an outcome entry on any change to a recorded file.
   - That makes a map "affected" even when the selector's verdict is "current", but D1 presents the three outcomes as the selector's verdict alone.

9. **LOW — D8: the first run is unstated.**
   - With no exit record, for example on a fresh machine, the trigger condition "HEAD differs from planned HEAD" is undefined. It should read as "start".

10. **LOW — D9 criterion wording.**
    - "with every other slot free but one, upkeep does not take a slot" is ambiguous.
    - The rule is that the take needs at least two free slots: one to take and one left free.

**Decisions walked**

- D1, body change to an uncovered linked function → re-stamp: met only if the marker is updated (finding 2).
- D1, covered function → drop T, write at H, measure T: met.
- D1, non-body change → miss and whole rebuild: met. With a failed rebuild, not met (finding 4).
- D2, commit target and own-file body change: met. A package the diff touches still runs whole under `gate-scope-coverage`'s D3.
- D3, B→F function set used by the selector and the self-check: met.
- D4, failing test → outcome entry, dropped on a recorded-file change, re-measured: met. Spawn-marked test: not met (finding 3).
- D5, failure after the plan already wrote the map at H: met for the gate; re-queue loop (finding 4).
- D6, toolchain bump → every map misses → full rebuild: met, given finding 2.
- D7, self-check on a rebuild: met. Audit: not met (finding 1). Retired key: met.
- D8, backlog with a busy pool → the process waits in D9 with no restart loop: met. Exceptions in findings 3 and 4.
- D9, three slots with one gate → take; second gate → yield within one poll: met. Hand call with the lock held: finding 6.
- D10, whole rebuild killed after round 1 → partial map at H, rest queued next pass: met.
- D11, anchor move → EndRole → finished rounds written: met once D10 has landed (finding 7).
- D12, the exit record feeds the D8 trigger: met, apart from the missing first-run record (finding 9).
- D13, carrier onto darn tasks: not met (finding 5).

No further tool calls.
