```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision on one concrete case and checked the end state against the decision's stated purpose.

**Decisions walked**

- D1, Current: a landing changes the body of a linked function no test of B covered. B is re-stamped to H and keeps every entry. Purpose met.
- D1, Affected, followed by an interrupted pass: T is dropped, the pass ends on a HEAD move, and the next plan gives a Current verdict. T's gap is never queued. Purpose missed (F1).
- D2: a body change to B's own non-test function selects the tests that covered it. Block keys are content-keyed (`internal/testcover/blockkey.go:22-32`), so kept entries still match. Purpose met.
- D3: a test of B that calls F in A is recorded as A's function set. Purpose met.
- D4, outcome entry: a body change to one of B's own functions that no test covered gives a Current verdict. D4 drops the outcome entry, but no verdict queues the test. Purpose missed (F1).
- D4, spawn-marked test: never run and never queued. Purpose met.
- D4, deleted test: its stale entry is not counted as a cover, because `judgeRuns` drops any entry whose `FuncHash` no longer matches (`internal/gatecli/redundancy.go:588`). Purpose met.
- D5: the build fails at H after a Miss. The old map at C is kept, the same H is not re-queued, and H2 retries. Purpose met.
- D6: a toolchain bump changes the stamp, so the gate misses and upkeep rebuilds. Purpose met.
- D7: an all-Current pass on a 3-slot hub with a free slot runs the audit. Purpose met; the bounds are stated.
- D8: a 1-slot hub where the plan queues work. The pass ends with its queue unworked, and the next tick starts it again. Purpose missed (F3).
- D9, yield on 3 slots: a pass and one gate hold slots, a second gate takes the last one, and the pass yields within one poll. Purpose met.
- D9, a hand call naming [A, P] on the 2-slot template while a pass holds P: deadlock. Purpose missed (F2).
- D10: two `t.Parallel()` tests each get their own profile, and a HEAD move discards only the current round. Purpose met. The ordering input exists as `PackageLaunch.ElapsedSeconds` (`internal/gateslot/runlog.go:113-117`).
- D11: an anchor move calls `EndRole` on `RoleTestMap` under `HubOwner`. The finished rounds are written. Purpose met.
- D12: the exit record carries the planned HEAD and the queued count, and D8 reads both. Purpose met.
- D13: every change maps to a Decision, and the follow-up is filed as a note. Purpose met.

**Findings**

**F1. MEDIUM · design · D1 (with D4, D8, D10)**

A test left with no entry is re-queued only when a later plan for its map gives an Affected verdict. A Current verdict queues nothing, so the gap stays until some later landing affects a test of that map.

Gaps come from:
- a pass ended by a HEAD move;
- a D10 round discarded on a HEAD move or a dirty linked file;
- a D4 outcome entry dropped on an own-file change that affects no test.

In that last case, D1's Current rule ("every entry is kept") also contradicts D4's drop rule. The gate stays sound, because a test with no entry still runs whole. But the upkeep goal fails for these tests: the backlog does not drain on its own. The D4 criterion "is queued again" holds only within the same pass.

Evidence: `_lym/discussion/decision-record.md:34-38`, `:92-93`, `:144`, `:184`, `:273`.

**F2. MEDIUM · design · D9 (with D1)**

A yielded pass and a hand call can deadlock until HEAD moves. Case: the template's `slots: 2` hub, with a pass building P while the hub is otherwise idle.

1. `lym gate testmap A P` builds A. A call keeps its slot for the rest of the call (`internal/gatecli/testmappackage.go:113-139`), so both slots are now held.
2. The pass sees every other slot held. It yields and releases its slot.
3. D9 does not say the pass releases P's build lock when it yields, and D1 rewrites a map only under that lock. So the pass keeps P's lock and waits for two free slots.
4. The hand call keeps one slot while it waits for P's lock (D9 says it waits). Only one slot is ever free, so the pass never retakes.

Neither side has a bound until a landing ends the pass. The redundancy sweep's hand call (D13) can hit this.

Evidence: `_lym/discussion/decision-record.md:42-45`, `:164-171`; `internal/gatecli/testmap.go:384-399` (lock taken before the slot, slot kept for the whole call).

**F3. MEDIUM · design · D8 (with D9)**

On a hub with fewer than two slots, passes restart in a loop. The D9 pass plans, re-stamps and ends with its queued packages unworked. Its exit record then names queued work, and D8 starts a pass whenever the record does.

D8's carve-outs from "queued work" cover dirty, failed-at-HEAD and spawn-marked packages only. So every dobby tick, at least once a minute, spawns a pass that plans every package through `CodeFileAt` and ends again. That misses D8's own purpose that nothing "restarts passes in a loop".

Evidence: `_lym/discussion/decision-record.md:141-147`, `:169`, `:176`.

**Tool calls**

- `internal/testcover/blockkey.go` 1–60 and a grep of `judgeRuns` in `redundancy.go`: to learn whether block keys survive an own-file re-stamp (D1/D2 walk). The manifest did not hold `BlockKey`.
- `grep judgeRuns/FuncHash` in `internal/gatecli/redundancy*.go`: to check whether a deleted test's kept entry counts as a cover (D4 walk). The manifest held only `mappedPackages`.
