---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:165-168
    summary: D8's current-row bound on join-parts is false; halted runs, a pre-resume goto and live part writers all pass it and can overwrite the plan directory
    fix: "In D8, replace the Bound and the current-row refusal so join-parts runs only while the run's status state is running at the Parts-Plan-Write row (read from the run status's state and current producer, never a row name literal) and only from the chair's strand, read from the caller's strand name against the table's chair; state that a halted run (blocked, failed, paused or awaiting) at that row, a run moved onto it by goto before resume, a part writer, the operator's shell, orch, a tackler and an arbiter are all refused, each refusal naming its way forward (for a halted run: lym loom goto --to Parts-Plan-Write then lym loom resume, whose fresh spawn rejoins). Change the sentence 'Each refusal names the part to fix and that its writer fixes it' so it names the part to fix and says the chair has its writer fix it, or fixes it itself when that writer no longer answers (D9). Add to D11 that a part writer never runs lym loom join-parts or lym loom insert-card. Extend the D8 acceptance criterion with refusals for a blocked run at the row and for a caller that is not the chair."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:133
    summary: D6 leaves a writer timeout unmapped and states a died or not-started writer stays an error carrying its own start error, which mapOutcome does not produce
    fix: "In D6, replace the sentence beginning 'A writer that failed to start or died stays an error' with a mapping for every way a writer seat ends the step: a writer that timed out maps to Stuck with its first output path as pointer and a reason naming the seat, its part and the timeout; a writer that failed to start maps to an error wrapping shuttleengine.ErrNotStarted carrying the writer's StartError; a writer that died maps to an error wrapping shuttleengine.ErrAgentDied naming the writer seat, not the chair. State that MultiLLMProducer reads the ended seat's SeatResult (EndedStep, Outcome, StartError) before the chair's result, so the stopped chair's outcome never decides these cases. Add each of the three cases to the D6 acceptance criterion."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:107-110
    summary: a resume during a fix round removes the done writer mid-fix and judges its half-edited part, telling the chair the part stands or ending the step, and the chair's fallback fix contradicts its cards-not-code limit
    fix: "In D5, after the resume paragraph, state that a done writer the probe removed is counted finished only when its outputs exist and Accept passes, and the line the chair is told says the writer was stopped at the interruption and any fix the chair had asked of it is lost. In D9's second bullet, state that a part whose fix request was outstanding at a resume is still open: the chair rechecks it against its contract and fixes it itself before running join-parts. In D12, state that when the chair fixes a part itself it may read the code of that part's area the fix needs, and add that to the chair stencil's reading rule. Add to the D5 acceptance criterion: a resume after a done writer was stopped mid-fix tells the chair the fix was lost."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:51
    summary: the chair runs on the same plan_timeout_min as each writer although it starts first and spans every writer plus fixes, join and shared spots, and a chair timeout fails the row uncommitted with a reason naming the chair
    fix: "In D2, replace 'Both rows take plan_timeout_min as their timeout' with: Parts-Write and every part writer take plan_timeout_min; the Parts-Plan-Write chair takes a new key parts_chair_timeout_min, default twice plan_timeout_min, hub-tunable and settable in a profile or run_settings. In D6, state that a chair timeout while a writer has already ended the step is decided by the writer's mapping (EndedStep wins), and that a chair timeout with no writer ended maps as Plan-Write's does today. Add the key to the D2 acceptance criterion's key list."
  - id: F5
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:199
    summary: the Parts-Write stencil tells the agent to run the parts check before ending its turn, but no decision gives it a command to run
    fix: "In D3, add a verb lym loom validate-parts that runs the partsparser check over _lym/parts/parts.md and prints its findings, read-only, refusing outside a run whose recipe holds a Parts-Write row, read from the recipe's rows, with a way forward. State that it is the parts gate's check set, one function for both. In D10, change the last bullet to run lym loom validate-parts before ending its turn and fix every finding. Add the verb to the PATTERN-loom-contract and PATTERN-refusal-way-forward constraint lines and to the D3 acceptance criterion."
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:82
    summary: D4 commits _lym/parts/ on Done only, while the NewPlanWrite decorator it copies and D6 commit on a gate-failed Stuck too, so an exhausted parts gate may halt with a dirty tree
    fix: "In D4, change 'that commits _lym/parts/ on Done' to 'that commits _lym/parts/ whenever the producer reports a non-empty output pointer, on Done and on a gate-failed Stuck alike'. In the D4 acceptance criterion, add: an exhausted parts gate leaves the run Stuck at Parts-Write with the parts directory committed and a clean tree."
  - id: F7
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:266
    summary: the first Open risks line asks for the stencil approval that the parent's Added after Discussion entry gave, and that entry says to drop it
    fix: "Delete the first bullet under ## Open risks, the one beginning 'The board asks Discussion to settle the stencils' content'."
  - id: F8
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:179
    summary: the chair's instruction 'never again after the next step begins' is ambiguous in a stencil where step also names the whole seat-engine step
    fix: "In D9's fourth bullet, replace 'and never again after the next step begins' with 'and never again once its first shared-spot card is in'."
  - id: F9
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:191-199
    summary: the Parts-Write stencil never tells the agent the parts_max cap or that every parts file carries a split note, both of which the parts check and format require
    fix: "In D10, add two bullets: 'hold at most parts_max parts, the value filled from config' and 'write the split note in every parts file: how the task splits, or why it does not'. Add parts_max to the markers the D10 acceptance criterion requires filled."
  - id: F10
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:267
    summary: the second Open risks line says the stencil forbids reading a neighbouring part's code, without the parent's signature-lookup allowance that now narrows that risk
    fix: "In the second Open risks bullet, after 'the stencil forbids it', insert 'beyond signature lookups with lym quarry glyphs or expand'."
---

### [BLOCKING] join-parts' current-row bound does not hold

**Location:** `_lym/discussion/decision-record.md:165-168` (D8), with D9 line 179 and D11 line 208.

**Issue:** D8 bounds `lym loom join-parts` by "only while `Parts-Plan-Write` is the current row".
The record states this guard as the verb's whole bound, and three cases break it.
The shed engine persists Blocked, Failed and Paused with `st.CurrentProducer` unchanged (`internal/shedengine/run.go:258`, `:383`, `:402`, `:416`, `:432`).

- **A halted run.**
  A Stuck at `Parts-Plan-Write`, which D6 commits, leaves that row current.
  So the operator, orch, a tackler or an arbiter can still run the verb.
  It overwrites the committed joined plan from the part directories and drops the chair's shared-spot cards and framing, leaving a dirty tree.
- **A goto before resume.**
  `lym loom goto --to Parts-Plan-Write` from a later halted row is D8's own way forward.
  It makes the row current before `resume`, while the part directories are not yet archived.
  A `join-parts` then overwrites the reviewed plan, and the fresh spawn's `NewPlanDirRotator` builds the chair's prior-plan amendment from the overwrite instead of from the reviewed plan.
- **Part writers.**
  Part writers are live inside the row.
  D11 forbids them only to edit the plan directory, never to run the verb.
  D8's refusal text even sends each fix to "its writer", so a writer that runs the verb after the chair's shared-spot cards are in discards them.
  D9 restrains only the chair.

Focus directive 2 answered: today every agent in the worktree can run the verb while the row is current, a halted or goto-moved run included.
It overwrites the whole plan directory.
The rubric's attack-surface rule requires a new verb's Decision to state what bounds it, and this Decision's stated bound is false.

### [MEDIUM] Writer timeout unmapped; died or not-started mapping misstated

**Location:** `_lym/discussion/decision-record.md:133` (D6).

**Issue:** `watch` ends the step for every end of a required seat other than an accepted done, `OutcomeTimeout` included (`internal/seatengine/engine.go` `watch`).
D6 maps two classes: a failed gate, failed Accept or missing outputs at resume go to Stuck, and "failed to start or died" stays an error.
A timeout fits neither class.

For the error class, D6 says the error "stays an error carrying its start error or wrapping `ErrAgentDied`".
`mapOutcome` reads only `result.Chair` (`internal/shedadapters/multillm.go` `mapOutcome`), and `join` has just stopped the chair.
So today's error describes the chair, never the writer's start error, which is the same defect D6 states for the Stuck case at line 134.

Focus directive 1 answered: for the gate and Accept cases, the record states the mapping that makes the outcome a Stuck naming the seat with parts committed.
For a writer that timed out, died or never started, it does not.

### [MEDIUM] Resume during a fix round loses the fix silently

**Location:** `_lym/discussion/decision-record.md:107-110` (D5), with D9 line 177 and D12 line 218.

**Issue:** `Probe` removes a done seat kept by its pane (`internal/seatengine/resume.go` `Probe` doc), even one busy on a fix the chair asked for.
D5 then counts that writer finished when Accept passes on whatever the interrupted fix left on disk.
The chair is told "its part stands", so its fix request is lost with no word.
If Accept fails instead, the step ends and the rerun is fresh, which archives every finished part (D6 line 137).

After any reinvoke under D13, no done writer answers, so D9's "the chair fixes that part itself" is the normal path.
D12 and the board still limit the chair to reading cards, not code, beyond the shared spots, which can leave it unable to make that fix.
Focus directive 3 answered:
- The live-chair resume rule holds for a part that was finished and untouched.
- The fresh-path archive matches D6's Bound.
- The hold-deferral change holds within one session; on resume, Probe removes the done writers it counts.
- The rule that a shared spot may use the first needing part's own export holds, since the chair places those cards.

### [MEDIUM] Chair timeout equals a writer's

**Location:** `_lym/discussion/decision-record.md:51` (D2), against D6 lines 131-135.

**Issue:** `Run` starts the chair first, and the chair then waits on every writer, runs fix rounds, the join and the shared spots, all under the same `plan_timeout_min` each writer has.
A slow writer lets the chair time out first.
`join`'s select takes `chairDone`, so `EndedStep` is never set, and `mapOutcome`'s timeout branch returns an error.
Nothing is committed and the reason names the chair, so D6's promise "a failed writer … leaves the run Stuck" is broken by a timing the defaults make likely.

### [MEDIUM] Parts-Write's self-check has no command

**Location:** `_lym/discussion/decision-record.md:199` (D10).

**Issue:** The parent's decision asks every review to check that each stencil instruction can be done with the tools the agent has.
D10 tells Parts-Write to "run the parts check before ending its turn".
D3 defines that check only as the `parts` gate's, run by Go, and no decision names a verb the agent can call.
D7 gives the part writer `validate-plan --part`, but Parts-Write gets nothing, so it either skips the instruction or spends gate attempts as its self-check.

### [LOW] D4 commits on Done only

**Location:** `_lym/discussion/decision-record.md:82` (D4) and the D4 acceptance criterion at line 275.

**Issue:** D4 copies `loomshed.NewPlanWrite`, which commits on any non-empty pointer, a gate-failed Stuck included (`internal/loomshed/planwrite.go:58-63`).
D6 states "on Done and on Stuck alike" for its own row, but D4 says "on Done" only.
Read literally, an exhausted `parts` gate halts with the archive moves and `parts.md` uncommitted.

### [LOW] Open risks keeps the line the parent said to drop

**Location:** `_lym/discussion/decision-record.md:266`.

**Issue:** The `Added after Discussion` entry at line 237 approves D10 to D12 and says "Drop the first Open-risks line, which asked for this approval".
The line is still there.
Under the rubric, an Open-risks line that a parent answer settles is a `consistency` finding whose fix drops the line.

### [LOW] "The next step" is ambiguous in the chair stencil

**Location:** `_lym/discussion/decision-record.md:179` (D9).

**Issue:** D12 carries D9's steps into the chair's stencil as instructions.
The chair's seat engine and its directives use "step" for the whole row, so an agent can read "never again after the next step begins" as allowing a rejoin until the row ends.
D9 line 184 shows the intended bound, the shared-spot cards.

### [LOW] Parts-Write stencil omits the part cap and the split note

**Location:** `_lym/discussion/decision-record.md:191-199` (D10).

**Issue:** D3 requires a split note in every parts file and refuses more parts than `parts_max`.
D10 lists neither: the cap reaches the agent only as a gate refusal, and the split note is mentioned only for the one-part fallback.

### [NIT] Open risks line 2 predates the signature allowance

**Location:** `_lym/discussion/decision-record.md:267`.

**Issue:** The parent's entry lets part writers look up signatures with `lym quarry glyphs` or `expand`.
The risk line still says the stencil forbids reading a neighbour's code, with no mention of that allowance.

## Rejected

- existing-paths finding 3, stale per-writer gate findings files: the lens could not find any reader of those files, and a findings file left from an earlier attempt that no code reads changes no outcome.
- interactions F4, mid-fix resume reported as a separate consistency finding: merged into F3, the same defect.
- walkthrough finding 2 and bound-breaker finding 3: merged into F2, the same unmapped writer ends.
- interactions F1 and F2 and interactions F6: merged into F1, one bound and the refusal text it governs.
