```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens report

**Decisions walked**
- D1, flag on, fresh run: Drift-Baseline → Parts-Write entered → Done → Parts-Plan-Write entered → Done → Plan-Write gate `!= true` false → Skipped → Plan-Bouncer. Purpose met (`shedengine/entry.go:30-77`, `shedbuild/entrygate.go:63-88`).
- D1, goto onto Plan-Write in a split run: goto sets `EnteredRow` and the gate is bypassed. Purpose met (`shedengine/goto.go:121-123`).
- D2, `parts_depth: heavy` in run_settings: the record asserts a load refusal but names no mechanism. Not a finding: I found no evidence either way.
- D3/D4, Parts-Write fresh spawn after an earlier split: the prep moves the parts dir entries, then `archiveStaleOutputs` finds `parts.md` already gone, then the seat runs. Purpose met.
- D5/D6, a writer's `plan-part` gate exhausted: the end state misses its purpose (finding 1).
- D6, building the writer seats' gate from `part_gates`: a step's input differs from what the step before it produces (finding 2).
- D6, fresh spawn with three parts: table built → probe → rotator and part-dir archive → `archiveStaleOutputs` → Run; the chair's inputs list overviews not yet written. Purpose met: the existing rule says every advisor output is one of the lead's inputs (`seatengine/table.go:138`).
- D7, part B using part A's `plan:` handle: dependency exports count as declared. Purpose met as decided.
- D8, part B using a shared spot's export: the join runs before the shared-spot cards exist, so the joined plan dangles until the chair inserts them. D9 orders `validate-plan` after the insert, so purpose met. D8's acceptance ("passes the plan gate's format checks") holds only for parts with no shared-spot export (finding 3).
- D9, chair writes `join.md` last: shuttle classifies Done on output existence, then the table's `plan` gate runs. Purpose met (`shedadapters/multillm.go:152-167`).
- D13, timing: `rowFamily("Parts-Write")` is `Parts`, a family the table lacks. D13 adds it. Purpose met as decided (`reportengine/timingspans.go:152-155`).

**Findings**

1. MEDIUM, design, D6 ("A Stuck from a failed writer or an exhausted gate halts the run") and D1's halt path. A failed writer never reaches Stuck.
   - The required seat's end fires `ender.end` (`seatengine/engine.go:130-136`).
   - `join` then stops the chair (`engine.go:361-370`). A stopped chair with no output records as stopped or died (`shuttleengine/stop.go:16-23`, `wait.go:1789-1790`).
   - `mapOutcome` turns `OutcomeDied` into an error wrapping `ErrAgentDied` (`shedadapters/multillm.go:169-182`). It reads no `EndedStep`, and nothing outside seatengine does either (grep of `EndedStep`).
   - So the run persists failed, as "chair died", without the writer's name or gate findings. The error can even read as transient (`shedtransient/shedtransient.go:32`).
   - The record states a Stuck halt as today's behaviour. Its fix is a decided mapping of `EndedStep` to Stuck, with a reason naming the seat and its findings.
2. LOW, design, D6 ("every writer seat carries the `plan-part` gate from the row's `part_gates` key").
   - Row gates are resolved once at construction from the row's Config, with no per-seat argument (`shedrecipe/entries_planwrite.go:27-66`, `entries_gate.go:46-48`).
   - A `plan-part` check needs the seat's part name and its directory (D7 `--part <name>`). The record doesn't say how one resolved entry is bound per seat.
   - The same gap holds for "keeps its gate findings at a path of its own": `KeepGateFindings` serves only the chair gate (`multillm.go:85-90`).
3. LOW, consistency, D8 acceptance ("numbers several parts into one sequence that passes the `plan` gate's format checks"). Taken literally, this can't hold for a part using a shared-spot export: the join runs before the chair's shared-spot cards exist (D9 order). The bullet should name the case it covers: parts using no shared-spot export.
4. NIT, consistency, D5 ("counts as finished when all its outputs exist and its Accept passes on them"). `Accept` takes a `shuttleengine.Result` (`seatengine/table.go:91`). A writer the probe didn't find has none, so the resume calls Accept with a zero Result. That works only because D7's part check ignores it, and the record doesn't say so.

**Tool calls**
1. `shuttleengine` GateSpec/GateEntry shape and plan-gate registration: needed for how a per-seat gate is formed.
2. `shedrecipe/entries_planwrite.go` and the `resolveGateSpec` heads: needed to see whether a row gate can be bound per seat.
3. Grep of `ValidateReworkPlan` users: to check whether a rework generation could reach the join numbering from 1.
4. Grep of `first_card` in the plan stencil, plus `prrework.go`: same question.
5. `find` of stencils and specs naming `first_card`: same question.
6. `first_card` lines in `contracts/specs/loom-plan-spec.md`: confirmed only the rework gate checks it. No finding.
7. Grep of loomengine config validation for D2's enum refusal: inconclusive, no finding. This call exceeded the six-call budget.
