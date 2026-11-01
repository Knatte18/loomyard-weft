---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/seatengine/resume.go:128
    summary: the seat engine has no test for a writer not found at resume whose outputs are missing or whose Accept fails, the branch that ends the step
    origin: lens:code-tests
    fix: In internal/seatengine/resume_test.go, add two subtests to TestEngine_Resume_Writers beside the existing lost-writer-passes-Accept subtest. The first resumes a table made with asWriters, with a live chair and one writer absent from LiveTable.Advisors and its output file not written. It asserts that the step ends without the chair being sent a notice, that the result's writer entry has EndedStep true, StrandGUID empty and Outcome empty, and that the writer's Accept hook was not called. The second does the same with the output written and an Accept hook that returns an error. It asserts EndedStep true and AcceptErr set to that error, with no notice sent to the chair. Add no production code.
---

### [MEDIUM] No seat-engine test for the lost-writer branch that ends the step

**Location:** `internal/seatengine/resume.go:128`

**Issue:** D5 says a writer seat not found at resume whose outputs do not all exist, or fail Accept, ends the step.
`Resume` implements this in the `case seat.Writer` branch, which calls `ender.end(lost)` on the `else` side.
`TestEngine_Resume_Writers` has two subtests: a found writer kept, and a lost writer whose outputs pass Accept.
Neither reaches the `ender.end(lost)` side, so breaking it (for example letting a writer with missing outputs count as finished) fails no test.
`TestMultiLLMProducer_MapsAWriterThatEndedTheStep` has a row for a writer lost at resume, but it feeds a hand-built `SeatResult{}` to the mapper, so it pins the mapping and not the engine producing that result.
The mapper's `StrandGUID == "" && Outcome == ""` test for this case depends on the engine leaving both empty, which no test checks.

## Rejected

- Fork code-correctness F1 (MEDIUM, chair-only check could refuse a numbered chair strand): the seat engine starts every seat under an explicit name and refuses a strand whose name differs (`checkName` at `internal/seatengine/engine.go:305`).
  Reed never renumbers an explicit name (`internal/reedengine/doc.go:197`), so the live chair's name equals the one `join-parts` computes.
- Fork code-correctness F2 (LOW, `writeJoinedPlan` deletes before writing): the overview is the sole signal that a plan directory is complete, so a partial write leaves a directory that reads as incomplete.
  The error names the rerun that rewrites the whole directory, and the stencil tells the chair to rerun on an error.
  This is the designed failure path and not a defect.
- Fork code-correctness F3 (LOW, `PartsTable` passes `plan_timeout_min` 0 through): a spec timeout of 0 means shuttle's `run_timeout_min` (`internal/shuttleengine/spec.go:211`), and `PlanTable` passes it the same way (`internal/loomengine/plan.go:120`).
- Fork code-affected-set (LOW, `join-parts` is not in the lightweight-wiring set): D8 and the amended decisions do not ask for it, and the fork found no defect, only that it could not tell whether the omission was deliberate.
  The verb reads the run's status and the shed paths that only the full wiring sets up.
- Fork code-tests F1 (MEDIUM, no test for the hold deferral of a done writer): `TestEngine_Run_ChairHoldDeferral` has the rows `an unended writer mid-turn defers` and `a done writer mid-turn still defers` (`internal/seatengine/engine_test.go:722-723`).
- Fork code-tests F3 (LOW, no test for a `writeJoinedPlan` write failure): this is the designed failure path rejected above, and a test for an OS write error adds no behavior the join promises.
- Fork code-tests F4 (LOW, non-chair caller row not visible): `TestJoinPartsVerb` has `APartWritersStrandIsRefused` and `ASessionWithNoStrandIsRefused` (`internal/loomcli/joinparts_test.go:62-63`).
- Fork code-tests F2 (LOW, lost writer branch) is kept as F1 above, at MEDIUM, after I confirmed that no `asWriters` use in `resume_test.go` reaches the ending branch.

## Handler checks that produced no finding

- Declared change: the `PATTERN.md` line and `pattern/PATTERN-sole-parsers.md` carry the declared `PATTERN-sole-parsers` text, and no commit carries a `Pattern-Departure:` trailer.
- Plan-Write's composed prompt: I expanded the four included blocks into `loom-template-plan.md` and compared the result with the template before the diff.
  The only difference was blank lines at block boundaries, which come from my expansion not trimming a block's leading comment and blank line.
  I did not run the real stencil loader.
- Stencils: the part writer, chair and parts stencils state their own rule for the three cut lines, name the quarry lookups the amended decision allows, never let the chair edit the parts file, and match the done and stopped notices the seat engine sends.
- Part scope: every widening in `planparser` and `planglyph` is guarded by `plan.Part != nil`, so a plan with no scope is checked as before.
- `_lym/landing/summary.md` is absent, so there is no change description to check.
