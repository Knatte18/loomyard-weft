---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    carries: F1
    location: _lym/discussion/decision-record.md:89
    summary: The reach turn fails at Darn-Write, whose verify entry is wrapped by a cold fixer on by default; the turn's failing result goes to that fixer, the wrap re-runs the gate, the re-run passes, and the writer never sees the note
    fix: >-
      In D3, add a bullet saying a reach turn's failing result goes to the writer alone.
      A row gate's fixer wrap, `gatefixer.WrapRowGate`, spawns no cold fixer for that result, as it spawns none for a forked failure its SkipForked closure reports.
      The bullet also states the bound: the bypass applies only to a result whose findings are the reach note, so every other verify failure still goes to the fixer first.
      In the same bullet, say how the turn's attempt count is kept under the wrap.
      The wrap's spawning closure runs the inner gate twice per shuttle arrival, so the count of failing results since the last pass must count shuttle arrivals, not inner closure calls.
      Add the Darn-Write case to the third D3 acceptance criterion: with `darn.gate_fixer` set, a reach turn's result spawns no cold fixer and reaches the Darn-Write writer's session.
  - id: F2
    severity: BLOCKING
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:268
    summary: The added entry on the mutation check order (F6) orders six writer edits, and none is applied; the Goal, Scope, D10, D11, D12, the refusal constraint, the D10 and D12 criteria and the Open risk still build the mutation step in this run
    fix: >-
      Apply the edits the entry "Mutation check order: trial before the mutation step (F6)" lists, without editing the entry itself.
      (1) Goal: build order becomes the reader and the reach note, then failure triage, then the mutation planner and `lym gate mutate` (D9, D11), then the trial, then the `mutation` step (D10) only if the trial pays.
      (2) D10: open with one sentence saying it is built only after the trial pays, in a later darn the parent opens from the D11 board note's decision.
      (3) D11: replace "prints D10's report" with prints the survivors, uncovered blocks and packages not judged that D9 finds, in the form D10 names.
      The board note asks the parent, from the trial's counts, whether to build D10 and list `mutation` in the template's diff keys.
      Replace the rationale sentence "Shipping the step unlisted keeps it from costing any gate before the trial says it pays." with one saying the step is not built before the trial says it pays.
      Rewrite or drop the sentence "The `gate.yaml` template lists `mutation` in no key, so no gate runs the step until a hub lists it." to match.
      (4) D12: keep `mutation_max_mutants` and `mutation_budget_min`.
      Remove `mutation` from the third bullet's report names, which leaves `reach` alongside `redundancy`, and say the acceptance of `mutation` and D10's refusal of `mutation` without `reach` come with D10's later build.
      In Constraints, narrow the `PATTERN-refusal-way-forward` bullet to the refusals of D6 and D8.
      In Scope In, mark the `mutation` report step and its rubric note as designed here and built after the trial.
      (5) Mark both D10 acceptance bullets as checked at D10's later build.
      Drop `mutation` from the D12 criterion's accepted report names.
      Reword the `Review:` criterion on the template to fit the new D11 note.
      (6) Delete the Open risks bullet "Order of the mutation trial (decision for the parent)", and record the ruling in support-log.md's Review rounds under a Round 2 entry.
prior:
  - id: F1
    status: open
  - id: F2
    status: resolved
  - id: F3
    status: resolved
  - id: F4
    status: resolved
    answer: The fixer's reason holds; D4 keeps such a function in the unreached list with a caveat naming the packages whose older maps may reach it, so it is never listed as plainly unreached.
  - id: F5
    status: resolved
  - id: F6
    status: resolved
    answer: "Settled by the added entry \"Mutation check order: trial before the mutation step (F6)\"; the writer edits that entry orders are carried as F2 of this round."
  - id: F7
    status: resolved
  - id: F8
    status: resolved
  - id: F9
    status: resolved
  - id: F10
    status: resolved
  - id: F11
    status: resolved
  - id: F12
    status: resolved
  - id: F13
    status: resolved
  - id: F14
    status: resolved
  - id: F15
    status: resolved
---

### [BLOCKING] The reach turn reaches Darn-Write's cold fixer, not its writer

**Location:** `_lym/discussion/decision-record.md:89` (D3, the reach-turn bullets and the rationale at line 100)

**Issue:** The fix for round 1's F1 makes the verify gate return a failing result whose findings are the reach note.
At Darn-Write, the failing result never reaches the writer.
The `verify` entry carries `fixer: darn.gate_fixer` (`contracts/recipes/darn-recipe.yaml:132-134`), whose default is `sonnet[medium]`.
`internal/darnengine/template.yaml:5` describes it as the fixer that "clears a failing gate's mechanical findings before the writer hears of them".
`gatefixer.WrapRowGate` (`internal/gatefixer/rowgate.go:58-72`) hands a non-passing, non-pending, non-terminal failure to a cold fixer, with the findings as its report.
It then runs the closure again and returns that verdict.
By D3's own rule, the re-run finds no unreached function that an earlier turn has not listed, so it passes, and the writer never sees the note.
That defeats D3's stated purpose: "the operator asked that the writer get the note from its own gate before review, so it adds the missing tests itself".
Each turn also spends a cold-fixer spawn.
D3's attempt-budget rule counts failing results since the last pass, but under the wrap the inner closure runs twice per shuttle arrival, so the count drifts from the entry's re-prompt budget.
Code-Burler-Fix's verify entry carries no fixer, and the weaver's batch gate has none, so those two channels hold.
The existing-paths, walkthrough and bound-breaker lenses each found this independently.
The handler reached it from the same recipe and template lines.

### [BLOCKING] The added entry on the mutation check order is not applied to the record's body

**Location:** `_lym/discussion/decision-record.md:268` (the Open risks bullet; also lines 7, 14, 212-219, 225-233, 239, 259, 301-302 and 305, and `support-log.md:85`)

**Issue:** The added entry "Mutation check order: trial before the mutation step (F6)" rules that no part of this task's first build builds D10.
It names six edits the writer makes, and none was made.
- The Goal at line 7 still ends with "then the mutation check".
- D10 at line 212 opens with no trial condition.
- D11 at line 225 "prints D10's report", and line 233 keeps "Shipping the step unlisted…".
- D12 at line 239 has the config check accept `mutation`.
- The D10 criteria at lines 301-302 are unmarked, and the D12 criterion at line 305 accepts `mutation`.
- The `PATTERN-refusal-way-forward` constraint at line 259 still lists D10's refusal.
- The Open risks bullet at lines 268-273 still says "This record builds the planner, the step and the verb in this run".
- The support log at line 85 still says F6 "is with the parent".

Plan-Write reads only the decision record, so a plan built from D10, D12 and their criteria as they stand would build the step the parent ruled out.
The rubric makes an Open risks line that an added entry settles a consistency finding whose fix drops the line.

## Rejected

New findings in this incremental round must be BLOCKING and sit on a changed hunk, so the lower-severity new items below are not kept:
- interactions 2, walkthrough F-W3 (D9 places mutants on a function moved verbatim between files, which D2 calls unchanged): LOW, below the incremental bar.
- walkthrough F-W2 (D8's walk runs past the run's merge base to the parent's tip): the walk still includes every landing in the failing tree, so it over-lists suspects without missing one. MEDIUM, below the incremental bar.
- existing-paths 2 (the weaver's `card <id>` entries share `batchGateClosure` and could read a `reach.json` an earlier `batch` call wrote): LOW, below the incremental bar.
- bound-breaker 2 (D3 says "inside a gate slot" while the `--map` run holds the larger of tier0's and tier1's slot asks): LOW, below the incremental bar.
- bound-breaker 3 (the linked set is read from a map that predates an import the diff adds): LOW, below the incremental bar.
- bound-breaker 4 (a HEAD that moves mid-refresh leaves packages unmeasured with no reason in the note): LOW, below the incremental bar.
- bound-breaker 5 (a changed line outside every block, such as a package-level declaration, is neither mutated nor reported): LOW, below the incremental bar.
- bound-breaker 6 (D1's bound names a moved HEAD as ending the run before its index write, though it ends only the package walk): NIT, below the incremental bar.
