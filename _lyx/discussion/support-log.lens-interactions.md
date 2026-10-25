```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked every pair of decisions that act on the same state, plus each acceptance criterion against the decisions it touches.

**Findings**

1. **BLOCKING — D3 × D5: the way forward from two `Drift-Await` halts loops back to the same halt.**
   - D5 halts on a dead capture whose record still says `running`, and names `lyx loom goto Drift-Baseline` to capture again.
   - On re-entry, D3 starts a capture only when the record is "absent or failed", and leaves a "running or passed" one alone.
   - So after the goto no new capture starts, and `Drift-Await` halts again.
   - The same loop happens when the tree moved before the first await with a `passed` record.
   - The acceptance criterion "starts no second capture while one is running or passed" makes it worse.
   - Fix: D3 should treat a `running` record whose process is gone as failed. For a passed record whose tree moved, it should either recapture or refuse with a different way forward.

2. **BLOCKING — D4/D6 × D7 and the "never a later capture" rule: nothing stops a main capture after code has changed.**
   - `lyx drift capture` is both the verb the detached child runs and, with `--side`, the agent's diagnosis verb.
   - D6 bounds only `--side`, by the run budget.
   - A plain `lyx drift capture` during `Drift-Verify`, or run by any agent after Webster, would back up and replace the trusted baseline with post-change output. The next compare would then bless the drift.
   - That breaks D7 ("nothing … refreshes a baseline") and D5 ("never falls back to a capture taken later").
   - Fix: D4 needs a bound. A main capture should refuse once `awaited_at` is set, or whenever the run's current row is not `Drift-Baseline`.

3. **BLOCKING — D3 × D6/D7: `lyx drift scope` has no stated bound, so it can switch off the check it feeds.**
   - D7 limits `accept` to the `Drift-Verify` row, but D3 sets no such limit on `scope`.
   - At `Drift-Verify` the agent could run `lyx drift scope --scope off`, and D6 would return Done.
   - It could also re-scope to `partial` with a larger skip list, which skips drift with no reason recorded. That gets around D7's per-test reasons.
   - The rubric's attack-surface rule applies here.
   - Fix: refuse `scope` outside `Drift-Baseline`, or once a capture has started.

4. **MEDIUM — D4 × D5: `Drift-Await`'s wait bound can run out while the capture is still waiting for a gate slot.**
   - D5 bounds the wait by "the capture's own timeout plus one minute".
   - D4's `timeout_min` covers only the command's run. The slot wait before it is unbounded and legitimate.
   - If every slot is busy longer than the timeout, `Drift-Await` gives up on a healthy capture, or its behaviour is undefined.
   - Fix: say when the bound starts, for example from a slot-acquired time kept in the capture record, or measure it differently.

5. **MEDIUM — D4 × D6: side captures share the main capture's mechanism, and the record doesn't say which D4 rules apply to them.**
   - D4's dirty-tree refusal: the agent's in-progress uncommitted edits would block diagnosis, and fixes are committed one at a time.
   - The capture record: does `--side` write `.lyx/drift/capture.json`? If it does, it overwrites the main record that `Drift-Verify` checks for a "passed capture".
   - The backup rename: it should apply to the side directories only.
   - Fix: state each of these for side captures.

6. **MEDIUM — D3 darn foreground × crash or resume: the darn re-entry path is undefined.**
   - In darn, `Drift-Baseline` runs the capture in the foreground. If the lyx process dies mid-capture, the record stays `running` with a dead pid.
   - On re-entry, D3 "leaves running alone" and only returns Done on a passed capture. So the row either waits forever or does something unspecified, and darn has no `Drift-Await` to catch it.
   - Same root cause as finding 1.

7. **LOW — D5 × D3/D4: the record doesn't say whether a recapture resets `awaited_at`.**
   - After `goto Drift-Baseline` replaces a failed capture, does the new record carry an old `awaited_at`?
   - Stating that a fresh capture record starts without `awaited_at` closes a bypass of D5's first-await tree check.

8. **LOW — D1 × D6: only `Drift-Baseline` checks that the commands are declared.**
   - D1's refusal for undeclared `capture`/`compare` sits on `Drift-Baseline` alone.
   - If `compare` is removed from config after the baseline, `Drift-Verify` has no stated refusal or way forward.

**Pairs checked**

- D3 re-entry × D4 backup rename: held.
- D3 re-entry × D5 way forward (dead `running`, moved tree on `passed`): broken (1).
- D3 darn foreground × crash on re-entry: broken (6).
- D3 scope verb × D6/D7 bounds: broken (3).
- D4 main capture verb × D7 no refresh and D5 no later capture: broken (2).
- D4 side capture × D4 dirty-tree check, capture record and backup: broken (5).
- D4 timeout × D5 wait bound: broken (4).
- D5 `awaited_at` × Webster reroute through Plan-Bouncer: held.
- D5 `awaited_at` × D3 recapture: unspecified (7).
- D5 tree equality × D3 writing `_lyx/drift` records: held, because `_lyx` is a symlink into the weft and outside the code tree.
- D2 ungated `Drift-Await` × flag turned off mid-capture: held.
- D2 flag off × the acceptance criterion that `Drift-Await` is Done: held.
- D6 × D7 accept, including accepting at a `Drift-Verify` halt: held, assuming a halted run's current row stays `Drift-Verify`.
- D7 accepted list × D3 partial skip list at compare and capture: held.
- D6 per-session budget × Finalize reroute re-entering `Drift-Verify`: held, with the budget resetting per session.
- D6 `drift-verify` gate × gate-slot's "no dirty tree" rule: held, since fixes are committed.
- D1 command checks × D6: gap (8).
- Acceptance criteria against D1–D7: the re-entry criterion conflicts with D5's way forward (1); the others hold.

**Tool calls**

- `ls -la _lyx` together with `git ls-files _lyx`: checked whether `_lyx` sits in the code tree, which D5's tree-equality check depends on. The context didn't establish this. The `git ls-files` part was a read-only git command, which the directive said not to run.
