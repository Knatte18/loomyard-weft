```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism on one concrete case and checked the end state against the decision's stated purpose.

**Decisions walked**
- D1, agent strand runs `lym gate test --gate card --card X`, and the tiers outlast 250 s. The runner and phase file come from the in-process run's tier start and end, and the replay works. **Held.**
- D2, an implementer strand with no `--wait` and no inherited slot. A nested line under card-verify has its strand stripped, and the operator has no strand. **Held.**
- D3, a front gets 75 and the agent re-runs within the grace. A dead agent's lease passes, and the runner cancels the in-process `runTest`. **Held for `lym gate test`, broken for card-verify (F2).**
- D4, strand `…:implementer-2` drops to `implementer` and is checked against the lifetime the claude engine resolves from the unnumbered `Spec.Role`. reed numbers only the strand name (`internal/reedengine/strand.go:243`). **Held.**
- D5, the `card` key with five earlier calls where one was abandoned in its last tier. **Broken (F1, F3).**
- D6, card-verify of a card whose `**Verify:**` line is `lym gate test --wait --tier 1 ./x`. **Broken (F2).**
- D7, the implementer's card gate is rendered without `--wait` and re-run with the same words, so it joins. The fixer's `fix_gate` and Go's `--wait` round call have different words, so both run, as they do today. **Held.**
- D8, the driver's stencil is unchanged. **Held.**
- D9, the implementer reads 75 with `done: false` as "run again". **Held.**

**Findings**

**F1 — MEDIUM, design, D5 (`decision-record.md` D5, "leaving out … an abandoned call and a refused one")**
The run records cannot tell an abandoned call apart, so it enters the median as a full call.
- Evidence: `internal/gatecli/run.go:479-497` (`report`) writes the cancelled tier's record with its exit code and only then halts.
- So a runner whose lease cancels it inside its last listed tier leaves a record for every tier in `TierList`, with a short wall time.
- `RunRecord` (`internal/gateslot/runlog.go:38-73`) has no field marking a cancelled or abandoned call, and D5 adds only a call id.
- The abandonment lives only in the runner's exit file, which `ReadRunRecords` never reads.
- A refused call writes no tier record, so it drops out with no rule needed.
- Fix: D5 states how an abandoned call is marked in the records, for example a new record field the runner sets on cancel, and records that the record fields hold no such mark today.

**F2 — MEDIUM, design, D6 with D1 and D3 (`decision-record.md` D6, and D1's phase file and `slot_free: false` answer)**
For a card-verify run, the runner cannot know the phase or the slot holders, and nothing in the record lets the lease cancel the line.
- A card's `**Verify:**` line is a `--wait` gate call (D7 keeps it). `cardverify.Runner.RunLine` runs such a line unslotted, as a child through `verifyrun.Run`, with `context.Background()`.
  - Evidence: `internal/cardverify/runner.go:46-49,138-145` and `internal/gatecli/cardverify.go:97-103`.
- The line's slot wait and tiers happen inside that nested `lym gate test --wait` process. So the runner cannot fill D1's phase file ("waits for slots or runs a tier and which"), or decide when to answer `slot_free: false` with holders.
- D6's "with its slot taken inside the runner" is also untrue for such a line, which takes no slot.
- D3's "cancels its call as a terminating signal does today" has no context to cancel through, because the line runs under `context.Background()`.
- Fix: D6 states what phase a card-verify runner reports, and that its lease cancel ends the line's process tree.

**F3 — LOW, design, D5 ("the median wall time of the latest five calls")**
D5 does not say which record fields give a call's wall time.
- `RunRecord` holds a per-tier `WallSec`, which excludes the slot wait, and per-tier `WaitStart`, `Start` and `End` (`internal/gateslot/runlog.go:50-53`).
- The front's wait that the break-even prices includes the slot wait.
- Summing `WallSec` underestimates the wait. The span from the earliest `WaitStart` to the latest `End` of a call id matches it.
- Fix: D5 names the span it takes.

**Tool calls**
- `internal/gatecli/cardverify.go` `runCardVerify` and `internal/cardverify/runner.go` `execute`: my context had the runner's doc lines but not how card-verify drives a line or which context reaches it.
- `internal/gatecli/run.go` `report` and `writeRecord`, plus `tierrun.go` cancel sites: my context lacked whether a cancelled tier still writes a run record.
