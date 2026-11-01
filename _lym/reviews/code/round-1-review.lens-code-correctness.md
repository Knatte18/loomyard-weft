```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff, one concrete case per function or branch.

**Cases run**

| Function or branch | Case | Result |
|---|---|---|
| `seatengine` `watch` | writer done, gate nil, Accept ok | correct: done line queued, no step end |
| `seatengine` `watch` | writer done, Accept error | correct: step ends, no line queued |
| `seatengine` `watch` | writer timed out | correct: step ends |
| `Run` | writer fails to start | correct: `ender.end(failed)`, outputs set, `writerExit` reads `StartError` first |
| `join` | chair ends first while a writer already ended the step | correct: writer is marked `EndedStep`, so `mapOutcome` picks it |
| `join` | normal success, no writer ended | correct: not marked |
| `Resume` lost writer | outputs exist and Accept passes | correct: one notice, step continues |
| `Resume` lost writer | outputs missing | correct: Accept not called, `ender.end`, `writerExit` gives Stuck "outputs were missing at resume" |
| `Resume` lost writer | outputs exist, Accept fails | correct: `AcceptErr` set, Stuck "did not pass the part check" |
| `Resume` found, already-done writer | attached | done line re-queued to the chair (benign) |
| `advisorWork.outstanding` | done writer, session mid-turn | counted (deferred), as D5 asks |
| `advisorWork.outstanding` | timed-out writer or nil handle | skipped, correct |
| `Table.Validate` / `validateWriterKinds` | mixed advisor and writer seats | refused, correct |
| `Table.Validate` | writer table whose lead lacks `seat-directive-writer-chair` | refused, correct |
| `writerExit` | each branch with nil `Gate` | `gateAttemptsPointer` and `gateSpansOf` are nil-safe (`gateattempts.go:66-80`), correct |
| `Config.PartsChairTimeout` | nil | `2 × PlanTimeout`, correct |
| `Config.PartsChairTimeout` | set, equal to plan timeout | refused, correct |
| `Config.PartsChairTimeout` | `plan_timeout_min == 0`, set value 5 min, `runTimeout` 0 | accepted, correct |
| `Config.PartsChairTimeout` | negative set value | refused, correct |
| `LoadTapestryConfig` | `plan_timeout_min == 0` with a set chair timeout | deferred to table build, as the shared decision says |
| `partsparser.Check` | self-dependency | flagged, correct |
| `partsparser.Check` | `archive-` prefix | flagged, correct |
| `partsparser.Check` | spot needer unknown | flagged, correct |
| `partsparser.Check` | spot whose `uses` come from a later part | flagged, correct |
| `partsparser.Check` | export claimed twice | flagged once, correct |
| `partsparser.Check` | record with no decisions | no panic |
| `partsparser.Validate` | absent record | error returned; reached before the parts-file check, so the parts file is never judged. Acceptable. |
| `planparser.JoinParts` | zero-card part | depends on `ParsePlan` refusing it |
| `planparser.JoinParts` | contiguous card numbering | renumbering is correct |
| `planparser.JoinParts` | failure inside `writeJoinedPlan` after removal | see F2 |
| `planparser.ExportMet` | `plan:` handle, respelled canonical form | matches through `refsMatch` |
| `planparser.ExportMet` | re-sign export | exact string compare only; fine for glyph exports |
| `checkJoinable` | two parts create the same path | refused, correct |
| `loomshed.JoinParts` | missing overview, failing part | refused with a way forward, correct |
| `checkPartPlan` | `DecisionRecordPath` cleared | correct; both pattern checks skipped on parts |
| `archivePartsEntries` | absent directory, only archives present, name collision | no-op / `-N` suffix, correct |
| `NewPartsPlanRotator` | `parts.md` kept, earlier archives untouched | correct |
| `joinPartsVerb` | halted at the row | refused with the resume way forward, correct |
| `joinPartsVerb` | no status found, or another row current | refused, correct |
| `joinPartsVerb` | non-chair caller | refused, correct |
| `joinPartsVerb` | caller name vs chair name | see F1 |
| `settledSegmentGuard` / `segmentWriter` | Plan-Write Skipped | names `Parts-Plan-Write`, correct |
| `settledSegmentGuard` / `segmentWriter` | Plan-Write Done or no history | names `Plan-Write`, correct |
| `validate-plan --part` | combined with another mode flag | refused, correct |
| `validate-parts` | `DecisionRecordPath` under lightweight wiring | set in the lightweight wiring (`wiring.go:275`) |
| `PartsPlanTable` | absent or malformed parts file | error with a way forward, correct |
| `PartsTable` | `plan_timeout_min == 0` | table `Timeout` 0, whereas `PartsPlanTable` resolves it through `PlanTimeout(runTimeout)`; I did not confirm what a zero timeout means downstream. See F3. |

**Findings**

- **F1 — MEDIUM, design** — `internal/loomcli/joinparts.go` `newJoinPartsDeps` (about line 270).
  The chair-only check compares `LYM_STRAND_NAME` with `agentname.Format(NameShortname, NameSlug, "parts-plan")`.
  The reed strand code numbers the role when a same-role strand is still held (`internal/reedengine/strand.go:243`, `agentname.NumberRole(role, held)`).
  After a rerun with a leftover held chair strand, the live chair's real name is the numbered form, and a legitimate join is refused as "not the chair".
  The prompt's `chair_name` is built the same way (`internal/seatengine/prompt.go:28`), so the chair is told the same unnumbered name.
  I did not read the reed numbering conditions to confirm when it triggers.
  Fix: take the chair's name from the name reed gave the live chair strand, or accept the numbered form of the role.

- **F2 — LOW, design** — `internal/planparser/join.go` `writeJoinedPlan`.
  It deletes every top-level card file and the overview before it writes the new ones.
  A write failure partway leaves a plan directory with no overview and a partial card set.
  The error message tells the caller to run the join again, so it is recoverable.
  Fix: write the new files first, or into a staging directory, then swap.

- **F3 — LOW, design (unverified)** — `internal/loomengine/parts.go` `PartsTable`.
  It uses `time.Duration(cfg.PlanTimeoutMin) * time.Minute` directly, so 0 is passed through.
  `PartsPlanTable` resolves 0 to the run timeout through `PlanTimeout(runTimeout)`.
  If a table `Timeout` of 0 does not mean "shuttle's run_timeout_min" downstream, Parts-Write gets a different timeout from what D2 says (`plan_timeout_min` for every writer).
  Fix: pass the resolved `cfg.PlanTimeout(runTimeout)` to `PartsTable`, or confirm that 0 means the run timeout.

**Tool calls**
1. One Bash combining four greps (`SeatRole`; `gateAttemptsPointer` and `gateSpansOf`; `DecisionRecordPath` in `loomcli`; `agentname.Format` call sites). My context lacked these definitions and the strand-name formation.
