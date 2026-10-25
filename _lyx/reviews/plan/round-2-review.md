---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/11-drift-verify-producer.md:54
    summary: Step 7 tells a resume from the row's own dirty-tree or test-gate Stuck by that Stuck's reason, but neither the run status nor its history holds a Stuck's reason at the resumed entry, so the verify gate's trigger cannot be implemented as written
    fix: >-
      Replace the reason-text test with a scope-record mark, written and read the way step 3 already handles halt_named and halt_named_at, so it needs no new symbol.
      In card 2-drift-records, add two fields to the ScopeRecord paragraph, gate_halt (the opening text of the dirty-tree or test-gate Stuck the row last returned) and gate_halt_at (the time it was returned), both written through UpdateScope.
      In card 11-drift-verify-producer, step 7, drop "whose reason, past any DecisionHaltPrefix, opens with one of those two halts' opening texts".
      Instead say the row writes gate_halt and gate_halt_at in the same UpdateScope call that precedes returning a dirty-tree Stuck (step 7) or an exhausted dirty-entry or verify-entry Stuck (step 10).
      An entry is the resume from that halt when gate_halt_at is set and Status's last history entry is a Stuck of this row whose latest time (LastAt when set, At otherwise) is at or after it, the same history test step 3 uses.
      Only that entry runs the told verify entry's gate once, and only when AttemptHead is found and differs from HEAD.
      The entry clears both fields before it runs the gate, and a gate or dirty-tree failure writes them again with the Stuck.
      Keep the halt opening texts as package constants that open the Stuck reasons for the operator's reading, but state that no code reads them back.
      In the TestDriftVerify paragraph, make the rows "a resume from the row's test-gate halt ..." and "a resume after the agent's verify entry spent its attempts" seed gate_halt_at and a last history entry that is a Stuck of this row, instead of a reason text, and add a row where a last history entry of another row, or one older than gate_halt_at, runs no gate.
---

### [BLOCKING] Step 7's resume test reads a Stuck reason the run never keeps

**Location:** `_lyx/plan/11-drift-verify-producer.md:54` (step 7, and the matching sentence in step 10 and the `TestDriftVerify` rows that depend on it)

**Issue:** Step 7 runs the told `verify` entry's gate "only on the resume from this row's own dirty-tree or test-gate Stuck, that is, `Status`'s last history entry is a Stuck of this row whose reason, past any `DecisionHaltPrefix`, opens with one of those two halts' opening texts".
A resumed entry cannot read that reason.
`shedengine.HistoryEntry` (`internal/shedengine/status.go`) carries `Producer`, `Outcome`, `Output`, `At`, `LastAt` and flags, and its `Reason` field is documented as the entry gate's text on a skipped entry, empty on every other entry.
A Stuck's reason reaches the status file only as `Status.Error` on the blocked halt (`OutputPointer.Reason` persisted as the halt's error), and the shed's resume write at the start of the next step (`internal/shedengine/run.go`, "Step 3b, the resume write") persists `StateRunning` with an empty error before it calls the producer.
So at the resumed `Call`, `Status.Error` is empty and the last history entry holds no reason; the discriminator between a dirty-tree halt, a test-gate halt and any other Stuck of the row is unreadable.
The decision record's own history test (D6, the `unruled` resume) names only the entry's row, outcome and time, which is why step 3 works and step 7 does not.

The consequence is that the one place a `Drift-Verify` agent's committed code meets the `drift-verify` gate after a halt has no working trigger.
An implementer would either run the gate on every entry that finds `AttemptHead` different from HEAD, which breaks the plan's own rule that a loom Finalize reroute entry runs no gate, or never run it on a resume, which lets a failing-test fix reach the compare and Done; the `TestDriftVerify` rows that seed a reason text in the history would pass against fakes and describe a status no run ever produces.

Round 1's F3 proposed the reason-keyed history test and the fixer applied it as proposed (round 1 fixer report, F3 and its self-check), so this raises a defect that fix introduced, not a disputed item.
The fix keeps the plan's intended behavior (first entry: `AttemptHead` absent, no gate; loom Finalize reroute entry: last history entry is another row's, no gate; resume from the row's own gate halt: gate runs once) and moves the discriminator onto a durable record the row writes itself.

## Rejected

- Focus 1 (the capture's named job): not a defect.
  `HoldNamedJob` is called only by the hidden `capture-child` (card 7), which `StartCapture` starts with `proc.DetachBreakaway` and releases as its last act after `RunCapture` returns.
  `RunCapture` holds no job (card 5), the darn `capture: wait` row and the loom breakaway-refused fallback run it in the step with no named job (cards 9 and 14), and `lyx drift capture`'s `proc.Detach` fallback starts only the child, which holds the job in its own process.
  No path puts a step process in the named job, so releasing it ends only the child and, through the nested jobs, its command; on Linux `StartTied` sets `Pdeathsig`, so a foreground command dies with its step.
  Card 13 terminates the same name `CaptureJobName` builds, from the pair's path, before the loop seize.
- Focus 2 (live-chair probe before any check): not a defect.
  Step 2 probes through `Deps.Seats.Probe` whenever the agent-session mark reads open, before the dirty check, the verify gate, the counted-capture check and the compare, and a live chair goes straight to the seat producer's own probe and `Resume`.
  The mark is opened by `ResetRuns` in the fresh-spawn hook, which runs before the seat starts, so a live agent always has its mark open.
  Steps 1 (scope off, no record, waived) read only the scope record and cannot coincide with a live agent.
- Focus 3, first entry and loom Finalize reroute entry: not a defect.
  A first entry has no `AttemptHead` and a reroute entry's last history entry is another row's, so neither runs the gate, which matches D6's "when the session committed code".
  The only defect on this trigger is F1's unreadable reason.
- Focus 4 (every `Extends` group): not a defect.
  Each listed ref gains a section, a row, a line or a list entry and none has an existing sentence rewritten: `docs/overview.md` (cards 1 and 7, a module-tree line), `pattern/PATTERN-test-speed.md`, `docs/dotnet-hub.md` and `internal/gatecli/doc.go` (card 3, a table row and example lines), `pattern/PATTERN-daemon-wakeups.md` (card 9, a list entry), `internal/proc/doc.go` (card 4, which holds no existing job sentence), `internal/shedrecipe/doc.go` (card 14, a new section) and the new `internal/driftengine/doc.go` (every card that extends it).
  The cards that reword existing prose, `docs/overview.md`'s recipe paragraphs and the loom and darn doc tails in card 14, and the `DecisionHaltPrefix` doc in card 11, already list their refs under `Edit`.
