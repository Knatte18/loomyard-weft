```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I checked every pair of decisions that act on the same state, and checked the acceptance criteria and constraints against them.

**Pairs checked**
- Added after Discussion (F6) × Goal build order: broken. The Goal still reads "then the mutation check", not "planner and `lym gate mutate`, then the trial, then D10 only if the trial pays".
- F6 × D10: broken. D10 has no opening sentence saying it is built only after the trial pays.
- F6 × D11: broken. D11 still "prints D10's report", and its rationale still says "Shipping the step unlisted keeps it from costing any gate".
- F6 × D12: broken. D12 still has the config check accept `mutation` as a report name.
- F6 × the D10 acceptance criteria: broken. Neither D10 bullet is marked as checked at D10's later build. The D12 criterion still has the check accept `mutation`.
- F6 × Open risks: broken. The "Order of the mutation trial" bullet is still there and still says "This record builds the planner, the step and the verb in this run".
- F6 × Scope "In": broken. Scope still lists the `mutation` report step and its rubric note as in scope for this build.
- F6 × the `PATTERN-refusal-way-forward` constraint: broken. The constraint still lists D10's refusal as this run's, but F6 moves that refusal to the later build.
- F6 × support log: broken. Round 1 still says "F6 … is with the parent", and the ruling is not recorded.
- D2 × D9 (moved functions): broken. D2 treats a function moved verbatim to another file as unchanged. D9 takes its changed lines from a line diff of the same base and head files, so the moved lines count as added and get mutants.
- D3 × D10 order (reach before mutation): held.
- D3 refresh × D9 freshness, including the prime case: held.
- D3 refresh × the redundancy step: held. The redundancy step reads only the hub maps (`internal/gatecli/redundancy.go:286`), and D3 writes only anchor maps.
- D1 (worktree map wins when its hashes match) × D3 refresh: held.
- D1 (only the prime's `--map` writes the index) × D3 (writes anchor maps, never indexed) and × D6 (writes nothing): held.
- D3 reach turn × D5 HEAD match, across a commit of new tests and an unchanged turn end that `Verify` skips: held.
- D3 reach turn × the darn failed-tests-alone path (`failedBefore`) and × the budget clock: held. A turn follows a pass, so no failed tests are kept.
- D3 turn counting × `GateEntry.Attempts`, counted as consecutive failures (`internal/shuttleengine/gate.go:73`): held. The next pass resets the count.
- D3 listed-function record × several weaver batches sharing the verify directory: held.
- D3 × D7 on a red tier (no refresh, triage from the stored maps): held, and the bound is stated.
- D7 triage lines × `gateslot.FailedTests`: held. Four-space lines read as output (`internal/gateslot/runlog.go:309`).
- D7 D8 command line × D8's required `--since`: held, through the `<last green commit>` placeholder.
- D6 × D8 argument refusals: held.
- D4 caveat × the D3 turn (a caveated unreached function still fires a turn): held, as intended.
- D10 match × D5 match: held.
- D12 (`reach` on `batch`) × D3 (weaver batch gate turn): held.
- `PATTERN-test-speed` change × D3 refresh: held.

**Findings**

1. BLOCKING, consistency, `_lym/discussion/decision-record.md:7` (also lines 210–219, 225–233, 239, 260, 268–273, 301–302, 305 and `support-log.md:85`)
   - Summary: The F6 entry orders six edits, and the writer applied none of them. The Goal, D10, D11, D12, Scope, the D10 and D12 criteria, the refusal constraint and the Open risk still describe building the `mutation` step, its config acceptance and its refusal in this run. That contradicts the ruling that no part of the first build builds D10.
   - Evidence:
     - `decision-record.md:7` reads "then the mutation check".
     - `decision-record.md:210` opens D10 with no trial condition.
     - `decision-record.md:225` says D11 "prints D10's report".
     - `decision-record.md:233` reads "Shipping the step unlisted…".
     - `decision-record.md:239` has the check accept `mutation`.
     - `decision-record.md:268–273` holds the Open risk bullet.
     - `decision-record.md:301–302` are the unmarked D10 criteria.
     - `decision-record.md:305` has the check accept `mutation`.
     - The F6 entry's edits (1) to (6) are at `decision-record.md:246`.

2. LOW, design, `_lym/discussion/decision-record.md:194`
   - Summary: D9 takes its changed lines from a line diff of D2's base and head files. A function D2 calls unchanged because it moved verbatim to another file still gets mutants, so cap-limited mutants go to code the diff did not change, and its survivors reach review.
   - Evidence:
     - `decision-record.md:60` is D2's moved-function rule.
     - `decision-record.md:189` is D9's "lines the diff adds or changes".
     - `decision-record.md:194` says the changed lines come from the same base and worktree files as D2.

Tool calls: none.
