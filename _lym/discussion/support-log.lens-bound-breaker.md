```yaml
lens: bound-breaker
outcome: reported
```

bound-breaker: I tested every bound, guarantee and code claim in the decision record and support log with a counterexample. Three claims break badly enough to block; six more need fixes.

**Findings**

1. **BLOCKING — D7 and D1's Bound: the audit has no trigger in steady state.**
   - D7 runs the audit only "when its queue is empty and it holds a slot".
   - D8 ends the process when its queue is empty, and starts one only on a HEAD move or queued work.
   - A pass where every map is re-stamped never takes a slot, so it never audits.
   - Counterexample: a quiet night with no landings, or a day of body-only landings that only re-stamp. No upkeep runs or holds a slot, so no entry is ever re-measured.
   - With the age limit retired, an entry the selector wrongly kept lives indefinitely. That breaks D1's Bound ("lives until its inputs change or D7's audit reaches it") and D7's claim to replace the age rule.
   - Fix: give the audit its own trigger, for example "audit due" in the exit record that the dobby reads, or a pass started on a quiet tick.

2. **BLOCKING — D13: the carrier timing does not fit the receiving tasks.**
   - D13 says the parent records changes "before that entry's run reaches Plan".
   - The board lists `testmap-cross-package`, `coverage-test-select` and `gate-scope-coverage` with recipe `darn`. A darn run has no Plan row; its Darn-Write writes code straight from the record.
   - `lym loom decision add --help` refuses `--amends-plan` on a recipe with no Plan-Write row.
   - `testmap-cross-package` is already at Darn-Write. For the other two, no run exists until the dobby starts one, and their Darn-Write may begin at once.
   - So "before Plan" never happens. The changes can reach `coverage-test-select` and `gate-scope-coverage` only after code writing has started, or not at all.
   - Fix: name the darn-recipe point, such as before the entry's Darn-Write starts, or by editing the stored record through `decision-record-replace`.
   - The ordering question is for the parent.

3. **BLOCKING — D8 and Constraints: retiring `test_map_every` removes an off switch with no reinterpretation.**
   - The template says `test_map_every: 0 never refreshes the coverage maps`, and `dobbyengine/testmap.go` treats 0 as never.
   - A hub that tuned it to 0 to disable upkeep has the key pruned by the retired-key walk, and from then on upkeep runs on every landing.
   - PATTERN-config-layers: "a meaning change ships a `configreg.Reinterpretation` … or flags it with a way forward".
   - The record only lists the key as retired and is silent on the 0 meaning.
   - Fix: keep an off switch (a key, or `slots`), or ship a reinterpretation that maps 0 to upkeep off.
   - This hub's base `gate.yaml` holds no `test_map_*` key, so this hub is unaffected; the hole is for other hubs.

4. **MEDIUM — D12 and D8: a hand call's exit record wipes the background backlog.**
   - Today `testMapRun.close` writes `ExitRecord` on every call, hand calls included, and there is one `testmap-exit.json` per hub.
   - Counterexample: the `redundancy-sweep-existing` sweep runs `lym gate testmap <pkg>` by hand while a backlog of 40 packages is recorded. The hand call's close writes a record with the planned HEAD at the current HEAD and queue 0.
   - D8's dobby then sees no work, and the backlog stalls until the next landing.
   - Fix: only the background pass writes planned HEAD and queue, or a hand call leaves them as they were.

5. **MEDIUM — D8 and D9: a waiting pass blocks re-planning, which needs no slot.**
   - The process stays live while it waits for a polite slot; D9 has no end to that wait.
   - D8 re-reads HEAD only "before each package job", and the dobby starts nothing while a process is live.
   - Counterexample: slots 3 with two gates running back to back, so the take that must leave one slot free is refused for hours. Meanwhile 15 landings arrive.
   - The plan, which re-stamps without a slot, never re-runs. Maps fall behind exactly when gates are busiest, and miss more often.
   - On a hub with fewer than two slots the process lives forever: D9's Bound says "never takes one", but D8 only ends the process on an empty queue.
   - Fix: re-plan, or end, when HEAD moves while waiting; end at once when the slot count rules upkeep out.

6. **MEDIUM — D4, D9, D10 and D5: a killed run or killed build reads as a failure.**
   - D9's yield and D11's `EndRole` kill runs in flight. `runproc.EndRole` sends SIGTERM, then SIGKILL, to the registered process and the processes below it.
   - A test process killed mid-run returns as a failed or timed-out run. Under D4 it gets an outcome entry, which is kept until a recorded file changes, so the test is never measured at this HEAD.
   - A yield during the coverage build makes the build fail. Under D5 that records a failed build at H, and the package is not retried at H.
   - Today's code avoids this: "A build killed by a terminating signal writes nothing".
   - Fix: state that a run or build ended by a yield or signal records no outcome and no failure.

7. **MEDIUM — D2 and D1 ("adds no new way to be wrong"): own-package non-Go files are compared by no rule.**
   - `testmap-cross-package`'s D2 hashes all files only for linked packages "other than its own"; the package's own `Files` are Go files. `coverage-test-select`'s D5 non-Go rule covers only a linked package's recorded set.
   - Counterexample: a landing edits `internal/b/testdata/golden.json` or an embedded file of B.
   - The plan sees no Go change and re-stamps B's map to H. Entries measured with the old file now claim H.
   - A gate in a worktree that does not touch B then trusts those entries.
   - Today's rule has the same hole, but the age limit bounded it at 7 days. D7 removes that bound and nothing replaces it for own non-Go files.
   - Fix: record the package's own non-Go files as D2 does for linked packages, and treat a change to them as a miss.

8. **MEDIUM — D1 and D5 against the format marker in `coverage-test-select`'s D5.**
   - That D5 makes "a marker whose `Commit` and `BuiltAt` differ from the map's own" a miss.
   - D1's re-stamp changes `Commit` with no build, and D5's failure write records H in a map that stays at C.
   - The record does not say the re-stamp also updates the marker's `Commit` and sets `BuiltAt` consistently, nor that the failure write leaves both alone.
   - Done naively, every re-stamped map misses at every gate.
   - Fix: state the marker rule for both writes.

9. **LOW — D10's discard check misses added files.**
   - "Reads the files the map records afresh" cannot see a file added mid-round, such as a new testdata file a test globs.
   - `testmap-cross-package`'s second dirty read covers untracked files in a dirty prime, but a landing adds a tracked file.
   - Fix: compare the recorded set and any added file of the linked directories.

**Claims that held**

- D1's soundness argument for a lagging task worktree: comparing from H against an older base only adds tests.
- D2's argument for own non-test files under the gate: a package the diff touches still runs whole.
- D2's test-file rule: Fuzz or Example changes fall into the miss branch, which is conservative.
- D6's stamp is stable across worktrees: `renderTierEnv` substitutes only `{parallel}` from hub-wide config, with no worktree paths.
- The Constraints claim that retired keys are pruned by the walk: `gateslot` already declares `RetiredKeys` and `configengine` handles them.
- D11 combined with D8's no-start-during-a-move rule.
- D9's yield bound of one poll, on 3 slots, against both a single gate and a multi-slot drain.
- D5's no retry at the same H, for a genuine failure.
- D4's spawn skip against `gate-scope-coverage`'s D3.
- The Constraints' code facts: the anchor-move wait, `Unbuilt` with no production reader, and the redundancy step counting spawn-marked tests as covers.

**Tool calls**

- Read `renderTierEnv`, to check whether D6's stamp would differ by worktree; the context did not have its body.
- Read gateslot's `RetiredKeys` and where `test_map_every` is used, to test the key-retirement claim and the meaning of 0; the context had only the agent's summary.
- Read `runproc.EndRole`'s doc and the anchor-move bound, to test D11 and the killed-run path; the context did not have the signal sequence.
- Read the hub's tuned `test_map` keys and slot count, and the template comments, to test the off-switch break; the context had no template comment text.
