---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:47
    summary: D2 and the Auto-mode assumption say every assistant line carries its message's final stop reason, but a message cut off mid-stream writes lines with a null stop reason, which D2 gives no named reading and no Acceptance row pins, nor any other unknown stop reason
    fix: In D2 (line 47), replace the sentence with one saying an assistant entry whose stop reason is absent or null, or is any value other than tool_use, end_turn or stop_sequence, reads unknown. In Auto-mode assumptions (line 85), qualify the assumption so it covers only a message that completes, and say that a message cut off mid-stream carries a null stop reason. In the claudeengine Acceptance criterion (line 101), add two rows, an assistant line with a null stop reason and one with an unlisted stop reason such as max_tokens, both reading unknown.
  - id: F2
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:57
    summary: D3 adds a recorded ForkSubagents to RunState for the gate, but the existing fork audit at finalize still reads the attaching Call's spec, so one attached run can get two answers to whether it was authorized to fork, and D3 leaves that unaddressed
    fix: In D3, after the sentence at line 58, add one sentence deciding whether the run's fork audit at finalize also reads the recorded authorization instead of the attaching Call's spec. If it does, add it to the RunState bullet in Scope In and add an Acceptance row for a twill-spawned run attached by a single Call that is still fork-audited. If it does not, say the audit keeps reading the attaching spec and why that is acceptable.
---

Focus 1 (D3, where the gate learns twill): verdict, not a defect.
`RunState` in `internal/shuttleengine/rundir.go` holds no fork authorization today, and Scope In's fourth bullet adds one, saying that a record without it reads as not twill.
On attach, `reconstruct` in `internal/shuttleengine/attach.go` keeps the candidate's persisted `state` and re-persists it with only the pid changed, but takes `spec` from the attaching Call.
So a field in `RunState` survives an attach by a later Call whose `plan_producer` reads differently, while `run.spec.ForkSubagents` would not.
D3 names `RunState` as the source, and the two attach rows in Acceptance pin both directions.
`PlanSpec` sets `ForkSubagents: twill` (`internal/loomengine/plan.go`), so the recorded value matches the twill decision at spawn.
F2 is the one leftover from this reading.

Focus 2 (D2's readings): I checked the stated readings against every main-chain assistant line in the 400 newest local transcripts (Claude Code 2.1.294–2.1.296).
Every split message's lines carry one stop reason (81 thinking lines and 19 text lines carry `tool_use`).
Every synthetic API-error entry carries `stop_sequence` with model `<synthetic>`.
Local slash commands land as `system` `local_command` entries, which D2 skips.
D2 states, and Acceptance pins, the split thinking or text line with `tool_use`, the `end_turn` thinking line, the interrupt marker and the API-error entry, each with one reading.
The gap is F1.

### [LOW] A null stop reason has no stated reading and the unknown branch is unpinned

**Location:** `_lyx/discussion/decision-record.md:47`, `:48`, `:85`, `:101`

**Issue:** 25 main-chain assistant lines in the local transcripts carry `"stop_reason": null`, in thinking, text and tool_use lines.
In 13 of them the next main-chain entry is `[Request interrupted by user]`: the message was cut off mid-stream and never got its final stop reason.
That disproves the premise at line 48 and the assumption at line 85 for that case.
Line 47's "any other stop reason reads unknown" does not clearly cover a missing or null value, so the plan writer has to guess how to read such a line.
The reading that fits D1 is unknown, which falls back to today's override.
No Acceptance row pins the unknown branch for an assistant line, so the stated reading is never tested.

### [LOW] Two readings of the fork authorization in one attached run

**Location:** `_lyx/discussion/decision-record.md:57`

**Issue:** `Run.finalize` in `internal/shuttleengine/wait.go` runs `AuditForks` when `run.spec.ForkSubagents` is set.
After an attach, `run.spec` is the attaching Call's normalized spec (`reconstruct`, `internal/shuttleengine/attach.go`).
With D3, a twill-spawned Plan-Write run attached by a later Call that reads `plan_producer` single would get the lens-review check from the recorded field but skip the fork audit.
D3 adds the recorded field but does not decide whether the audit should switch to it.
The plan writer either changes the audit with no decision behind it or leaves the disagreement in place without knowing it is there.
