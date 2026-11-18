---
verdict: APPROVED
findings:
  - id: F12
    severity: LOW
    class: consistency
    location: _lym/plan/13-gate-mutate.md:113
    carries: F6
    summary: The mutation_budget_min row of TestGateMutate names only a short minuteUnit as its input, but the coverage build shares that unit through redundancy_build_timeout_min, so the build times out before the mutants are planned and the row cannot reach its outcome
    fix: In card 13's Tests, extend the two budget-row bullet for mutation_budget_min so the row's hub config sets redundancy_build_timeout_min far above the real coverage build time (for example 100000) beside mutation_budget_min 1 and a minuteUnit so short that the budget is spent at the hold. State that the build bound and the budget share the one minuteUnit, so only the config values separate them. No target changes, so validate-plan --overlay has nothing to check.
prior:
  - id: F1
    status: resolved
    answer: 'Card 13 edits internal/gateslot/doc.go and names lym gate mutate as the third one-slot holder, which PATTERN-gate-slot covers through its rule of the non-tier holders the doc lists. Card 5 edits internal/gateslot/doc.go and pattern/PATTERN-gate-slot.md for the reach refresh, beside the map run.'
  - id: F2
    status: resolved
    answer: 'Checked card by card. Cards 1 and 2 build internal/testreach before any main imports it, and card 5 first imports it with the callers of every function cards 1 and 2 create. WriteIndex lands in card 6 with runTestMap, the reach report reader and matcher in card 7 with the loomcli wiring, RerunMark.Run in card 8 through spawnAndJudgeAgain, RerunMark.On in card 9, FuncsReachedBy and FirstParentCommits in card 10 with runReach, TriageLines in card 12, and ChangedLines and PlanLineMutants in card 13 with runMutate.'
  - id: F3
    status: resolved
    answer: 'Card 4 now lists internal/loomcli#gateConfigWithTwoTiers under Edit and states the changed matched and replacement lines. The exact line is at internal/loomcli/landingdeps_verify_test.go:36. The inline strings at internal/gateslot/config_test.go:806 are test input and not template text.'
  - id: F4
    status: resolved
    answer: 'Card 4 extends the bounds paragraph of internal/gateslot/doc.go with the three keys. Cards 5 and 13 add their runs to its runs-that-take-the-bounds sentences and to the docs/overview.md sentences.'
  - id: F5
    status: resolved
    answer: 'Card 10 gives reachRequest a limit field, which reachCmd sets to the 200-commit constant, and its range-cap TestGateReach row drives runReach with a limit of 2 over the three landings.'
  - id: F6
    status: open
    answer: 'The outcome past mutation_budget_min is stated and both rows exist. The mutation_budget_min row does not reach its outcome with the named input alone, as F12 says.'
  - id: F7
    status: resolved
    answer: 'Card 2 states that an absent sharedDir or worktreeDir reads as no maps and refuses nothing, and TestReader has the row.'
  - id: F8
    status: resolved
    answer: 'Card 9 logs a failure to record at Warn and returns inner pass without the turn, and its TestWithReachTurn row asserts it.'
  - id: F9
    status: resolved
    answer: 'Card 7 renders the prime reason for a Prime report before any match, and TestRenderReachReportNote has rows for a prime report and an unreadable one.'
  - id: F10
    status: resolved
    answer: 'Settled by the added entry "Triage in the prime follows D7 (plan F10)". Card 11 makes mergeBase answer no parent branch in the prime, and the overview Shared Decisions bullet takes no base for triage lines there.'
  - id: F11
    status: resolved
    answer: 'Card 13 lists internal/testcover#site under Edit and names its new callee field.'
---

### [LOW] The mutation_budget_min row cannot reach its outcome with a short minuteUnit alone

**Location:** `_lym/plan/13-gate-mutate.md:113`

**Issue:** Card 13 has `diffMutation` build each judged package's coverage binary whatever the budget, bounded by `redundancy_build_timeout_min`, and makes the `mutation_budget_min` row of `TestGateMutate` drive `mutateCmd` with a `minuteUnit` so short that the budget is spent at the hold.
`mutateCmd` mirrors `redundantCmd`, where `minuteUnit` scales both the budget and the build bound (`internal/gatecli/redundant.go:255`, `internal/gatecli/redundancy.go:462`).
The template defaults are a build bound of 10 minutes and a `mutation_budget_min` of 20 (`internal/gateslot/template.yaml:5`), so with a unit short enough to spend the budget at the hold, the real coverage build exceeds its bound first.
The build then fails, no plan exists, and no mutant is left to report unjudged with the budget reason.
The row reaches its outcome only if its hub config also raises `redundancy_build_timeout_min`, which the card does not say.
The `reach_budget_min` row has no build to time out, since the spent deadline starts none, so it reaches its branch as written.

## Rejected

- Card 13 reuses "card 5's refresh" while card 5 names no separate refresh unit; the refresh sits inside `reachStep.run`, and card 13 lists only the `reachStep` type under Uses.
  An implementer extracts the method inside gatecli, so this blocks nothing, and an incremental round raises a new finding only at BLOCKING.
- Card 11's prime clause of `mergeBase` has no `TestTriageLookup_Lines` row of its own, since the table scripts `mergeBase`.
  The no-parent-branch row covers the lookup's output, the clause is one closure line, and the finding would be below BLOCKING.
- Card 3's guard tier, the pinned files and the single-package `testcover.ReadMap` callers match the tree: the three pinned files are the only non-test callers outside `internal/testcover`, and the existing boundary guards are tier0.
