---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D6 (A Stuck from a failed writer), D1 (Neither new row carries on_stuck), Acceptance criteria D5 and D6
    summary: A failed writer does not halt the run with a Stuck today; the required seat's step end stops the chair, which mapOutcome turns into an ErrAgentDied error, so the run fails uncommitted with a reason that names neither the writer nor its findings
    fix: In D5 or D6, add a decision that MultiLLMProducer maps a result whose step was ended by a seat (a SeatResult with EndedStep set) to shedengine.Stuck with a non-empty output pointer, so the commit decorator commits the part directories, and a Stuck reason naming the seat, its part and its gate findings path or Accept error. Cite internal/seatengine/engine.go join (the chair is stopped on ender.signal), internal/shedadapters/multillm.go mapOutcome (OutcomeDied becomes an ErrAgentDied error, and EndedStep is never read) and internal/loomshed/planwrite.go Call (commits only on a non-empty pointer) as the evidence. Rewrite D6's sentence to state that mapping, and add to the D5 or D6 acceptance bullet that a writer's exhausted gate or failed Accept leaves the run Stuck at Parts-Plan-Write with a clean tree and a reason naming the seat.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D8 (It refuses outside a run whose recipe holds a Parts-Plan-Write row; Bound)
    summary: lym loom join-parts is bounded only by the recipe's rows and the files existing, so any agent of the run at any later row (a Plan-Burler-Fix fixer, Webster) can run it and silently replace the reviewed plan's cards, dropping the chair's shared-spot cards and the review's fixes
    fix: In D8, add a refusal that lym loom join-parts refuses unless the run's current row is the recipe's Parts-Plan-Write row (read from the run status's current producer and the recipe's rows, never a row name literal), with a way forward naming lym loom goto --to Parts-Plan-Write. Restate D8's Bound to say what the verb can now overwrite (only the plan directory, only while Parts-Plan-Write is the current row) and that a run past that row cannot reach it. Add to the D8 acceptance bullet that join-parts refuses while another row is current.
  - id: F3
    severity: BLOCKING
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md Open risks line 1; board entry plan-split-producers, section Stencils (operator, 2026-11-01)
    summary: The board requires Discussion to settle the content of the Parts-Write, part-writer and chair stencils with the operator before Plan; this autonomous run picked D10, D11 and D12 itself, and only the parent can say whether the operator approves them before Plan starts
    fix: Route to the run's parent and ask whether the operator approves D10, D11 and D12 as written, or which changes they want, before Plan starts. Once the answer is recorded (lym loom decision add, or edits to D10 to D12), drop Open risks line 1.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D6 (Bound, a resume keeps every finished part), D5 (On resume)
    summary: The keep-finished-parts resume runs only when the chair is live; a lost chair makes Probe stop every writer, and the fresh path's D6 preparation archives every part directory, so a chair crash, timeout or machine restart starts every part over, a case the bound never states
    fix: In D6's Bound, add that when the chair is not live at a reinvoke or rerun, Probe stops every writer and the fresh spawn archives every part directory, so every part, finished or not, is written again. Cite internal/seatengine/resume.go Probe and Resume (Resume needs a live chair) and internal/shedadapters/multillm.go Call (the fresh path runs prepareFreshSpawn). In D5, limit the resume rule to a resume with a live chair.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D5 (The chair's hold deferral counts writer seats as it counts advisors), D9 and D11 (the post-done fix loop)
    summary: advisorWork.outstanding skips every ended run, and a writer that reached done has ended, so a chair waiting on a fix it asked of a done writer is not deferred and notifies the parent on every contract-mismatch round
    fix: In D5, decide how the chair's hold deferral counts a done writer whose pane is kept, for example that a writer seat whose run ended done still counts as outstanding while its session reads mid-turn, read the same way outstanding reads a live advisor. Cite internal/seatengine/work.go outstanding (run.ended() is skipped) as the evidence. Add to the D5 acceptance bullet that the chair's held turn sends the parent nothing while a done writer is mid-turn on a fix.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D3 (Shared spots, the parts check), D9 (shared-spot card placement), Open risks line 3
    summary: A shared spot records its exports and the parts that need it, but nothing it uses from parts, so the parts check cannot refuse a spot that no card position satisfies (needed by an earlier part, using a later part's export), and the chair, barred from rejoining, meets it only as an exhausting uses-later-target finding
    fix: In D3, add to each shared spot the exports of parts it uses, and add a parts-check refusal for a shared spot that uses an export of a part that comes at or after the first part needing it, with a way forward to reorder the parts or move the dependency into a part. Restate Open risks line 3 to cover only placement within the now-guaranteed valid range, or drop it if D9 then states the bound.
  - id: F7
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D6 (every writer seat carries the plan-part gate from the row's part_gates key; keeps its gate findings at a path of its own)
    summary: A row's gates resolve once at construction with no per-seat argument, and KeepGateFindings serves only the chair, so the record does not say how the one part_gates entry is bound to each seat's part name and directory or where each seat's findings path comes from
    fix: In D6, state that the table source builds each writer seat's GateSpec from the resolved part_gates entry with the seat's part name filled in (the check running validate-plan --part <name>), and that each seat's findings path is _lym/parts/<part>/ plus a fixed file name the table source sets.
  - id: F8
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D1 (names the plan writer this run ran, read from its history)
    summary: After D1's own way back, a goto onto Plan-Write in a split run, the history holds both a Skipped and a later Done entry for Plan-Write, and the rule as written still names Parts-Plan-Write
    fix: In D1, change the rule to read Plan-Write's latest history entry, naming Parts-Plan-Write when that latest entry is Skipped, and Plan-Write otherwise. Extend the D1 acceptance bullet with the goto case.
  - id: F9
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3 (part name valid under seatengine.ValidateSeatName), D4 and D6 (except earlier archives)
    summary: The seat-name grammar admits a part named archive-x, whose directory reads as an earlier archive under the D4 and D6 preparation, so its stale cards are never archived
    fix: In D3, add a parts-check refusal for a part name that begins with the archive directory prefix, with a way forward to rename the part.
  - id: F10
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D5 (counts as finished when all its outputs exist and its Accept passes on them)
    summary: Seat.Accept takes a terminal shuttleengine.Result, and a writer the probe did not find at resume has none; the record does not say what Accept receives there
    fix: In D5, state that the resume calls the writer's Accept with a zero Result and that the D7 part check reads only the part directory on disk, or change Accept's signature and say so.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md Acceptance criteria D8 (numbers several parts into one sequence that passes the plan gate's format checks)
    summary: The join runs before the chair's shared-spot cards exist, so a joined plan whose parts use a shared spot's export cannot pass the plan gate's format checks at that point
    fix: In the D8 acceptance bullet, say that the numbered sequence passes the plan gate's format checks for parts that use no shared-spot export.
  - id: F12
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md D7 (--part is a fifth mode, exclusive with the four flags)
    summary: validate-plan already documents five modes (no flag, --require-approved, --rework, --lens-review, --overlay), so --part is a sixth
    fix: In D7, change a fifth mode to a sixth mode.
---

### [BLOCKING] A failed writer ends the run as an error, not as a Stuck

**Location:** `_lym/discussion/decision-record.md` D6, D1, Acceptance criteria D5 and D6

**Issue:** D6 says a Stuck from a failed writer halts the run.
D5 makes a writer behave like a required advisor.
When a required advisor ends without being accepted, `seatengine`'s `join` stops the chair (`internal/seatengine/engine.go:361-370`).
The stopped chair has no `join.md`, so it waits out as died.
`mapOutcome` turns that into an error wrapping `ErrAgentDied` (`internal/shedadapters/multillm.go:169-183`), and nothing outside `seatengine` reads `EndedStep`.
The run therefore persists as failed rather than Stuck, with a reason naming the chair's death and not the writer's gate findings.
The commit decorator commits only on a non-empty pointer (`internal/loomshed/planwrite.go:70-72`), so the part directories stay uncommitted.
`ErrAgentDied` can also be classed as transient (`internal/shedtransient/shedtransient.go:32`).
No production table has set `Required` yet, so this path has never run.
The acceptance criteria as written would pin behaviour the code does not produce.

### [BLOCKING] `join-parts` has no bound on when it may run

**Location:** `_lym/discussion/decision-record.md` D8

**Issue:** D8 refuses `join-parts` only on a recipe without the row and on missing files.
After Parts-Plan-Write is Done, both conditions still hold.
So a Plan-Burler-Fix fixer or a Webster session can run the verb and rewrite the plan directory.
That drops the chair's shared-spot cards and every review fix.
D9's "never again after the next step begins" is a stencil instruction to the chair, not a guard.
The Bound names the damage a rerun does but nothing that prevents it, which the rubric's attack-surface question treats as an unbounded verb.

### [BLOCKING] The stencils' content needs the operator before Plan

**Location:** `_lym/discussion/decision-record.md` Open risks line 1; board entry `plan-split-producers`, Stencils

**Issue:** The board says the three stencils are the heart of the task and that Discussion settles each one's content with the operator before Plan.
This run was autonomous, so D10 to D12 are the writer's own picks.
The Open risks line records that, but the writer cannot settle it, and no board note titled `Decision for plan-split-producers:` answers it.
Whether Plan may start on these stencils is the parent's question.

### [MEDIUM] Losing the chair discards every finished part

**Location:** `_lym/discussion/decision-record.md` D6 Bound, D5

**Issue:** `Resume` needs a live chair (`internal/seatengine/resume.go`).
With no live chair, `Probe` stops every writer, and `Call` takes the fresh path (`internal/shedadapters/multillm.go` `Call`).
On that path D6's preparation archives every part directory.
A chair crash, timeout or machine restart therefore rewrites every part.
The Bound states only the case of a writer lost before it finished.

### [MEDIUM] The chair's hold deferral ignores a done writer fixing its part

**Location:** `_lym/discussion/decision-record.md` D5, D9, D11

**Issue:** `advisorWork.outstanding` skips every run that has ended (`internal/seatengine/work.go:44`).
A writer that ended done still keeps its pane and, per D9 and D11, fixes its part when the chair asks.
While the chair waits on such a fix, nothing counts as outstanding.
So the chair's held turn reaches the parent as a stall on every contract mismatch.

### [MEDIUM] Shared spots have no ordering the parts check can verify

**Location:** `_lym/discussion/decision-record.md` D3, D9, Open risks line 3

**Issue:** A shared spot carries its exports but not what it uses from parts.
A spot needed by part A that uses an export of a later part B has no valid card position.
The parts check cannot see this.
The chair cannot rejoin, so its only signal is `uses-later-target` on the joined plan, which can exhaust the `plan` gate.
Open risks line 3 covers misplacement, not the case where no valid position exists.

### [LOW] Per-seat binding of the `plan-part` gate is unstated

**Location:** `_lym/discussion/decision-record.md` D6

**Issue:** Row gates are resolved once from the row's config (`internal/shedrecipe/entries_planwrite.go`).
`KeepGateFindings` serves only the chair.
The record does not say how each writer seat's gate gets its part name, or where each seat's findings path comes from.

### [LOW] The settled-verdict writer rule misreads a goto-recovered run

**Location:** `_lym/discussion/decision-record.md` D1

**Issue:** After `goto --to Plan-Write` in a split run, Plan-Write's history holds a Skipped entry and then a Done entry.
"Recorded Skipped" stays true, so the refusal names Parts-Plan-Write although the plan now comes from Plan-Write.

### [LOW] A part named with the archive prefix escapes archiving

**Location:** `_lym/discussion/decision-record.md` D3, D4, D6

**Issue:** `[a-z][a-z0-9-]*` admits `archive-x`.
Its directory is then skipped as an earlier archive, so its stale cards read as this session's output.

### [LOW] Accept has no Result to judge at resume

**Location:** `_lym/discussion/decision-record.md` D5

**Issue:** `Seat.Accept` takes a `shuttleengine.Result` (`internal/seatengine/table.go:89-91`).
A writer the probe did not find has no terminal result, and the record does not say what the hook receives.

### [LOW] D8's acceptance bullet cannot hold for parts using a shared-spot export

**Location:** `_lym/discussion/decision-record.md` Acceptance criteria D8

**Issue:** The join runs before the shared-spot cards exist, so a joined plan whose parts use their exports dangles until the chair inserts them.

### [NIT] `--part` is a sixth mode

**Location:** `_lym/discussion/decision-record.md` D7

**Issue:** `validate-plan`'s help already names five modes (`internal/loomcli/validate.go:164-166`).

## Rejected

- No fork finding was rejected. Where lenses reported the same defect at different severities, the consolidated severity follows the evidence:
  - The failed-writer path was rated BLOCKING because it falsifies the failure premise and the acceptance criteria.
  - The deferral gap was rated MEDIUM because it recurs on every fix round.
- The walkthrough lens's D2 note, that `parts_depth` names no refusal mechanism, was raised by that lens as no finding and stays none: refusing an out-of-set config value is the plan's call-site work, not a fact the design rests on.
- Leanness: no relocation finding. Each Rationale line and the D4 quarry evidence sentence carries a premise or the reason for a decision the plan writer needs.
