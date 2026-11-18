---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: The reach refresh holds the map run's slot ask, not one slot, so the gateslot doc names it in the bounds paragraph beside the map call's slot ask instead of in the one-slot non-tier holder paragraph; lym gate mutate joins that holder paragraph as proposed.
  - id: F2
    disposition: fixed-differently
    reason: Each production function now lands in the card of its first main caller by moving it there, not by folding cards 8 into 9 and 11 into 12; card 13 is folded into 14 as proposed, and the index card moves after the reach step, carrying WriteIndex and the guard's WriteIndex pin, which card 4 could not hold before that call exists.
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: ruling
    reason: Parent decision 'Triage in the prime follows D7 (plan F10)' carried out in card 11 (now 11, failure-triage) and the overview's Shared Decisions.
  - id: F11
    disposition: fixed
---

# Round 1 fixer report — plan

Two renumberings changed the card numbers: `remove-card` folded old card 13 into old card 14, and the index card moved from 3 to after the reach step.
The new order:

| New | Card | Old |
| --- | --- | --- |
| 1 | testreach-changed-funcs | 1 |
| 2 | testreach-reader | 2 |
| 3 | pattern-coverage-map-reader | 4 |
| 4 | gate-config-reach | 5 |
| 5 | reach-step | 6 |
| 6 | map-run-writes-index | 3 |
| 7 | reach-review-note | 7 |
| 8 | rerun-mark-writer-only | 8 |
| 9 | reach-turn | 9 |
| 10 | reach-verb | 10 |
| 11 | failure-triage | 11 |
| 12 | triage-in-fixer-reports | 12 |
| 13 | gate-mutate, which now includes old card 13 | 14 |

The tools rewrote every card-number reference.
Below, card numbers are the new ones.
`lym loom validate-plan` passes with no blocking finding.

## Changes

- **F1** (fixed differently): card 5 lists `internal/gateslot/doc.go` and `pattern/PATTERN-gate-slot.md` under **Edit:**.
  - `PATTERN-gate-slot`'s Gate slot section gets a new place beside the map run: a refresh of a task worktree's anchor maps, which runs in the map run's slot ask. Its lead bullet stops counting the places.
  - The gateslot doc names the refresh in its bounds paragraph, where it says a map call holds the larger of tier0's and tier1's slot asks.
  - I didn't add the refresh to the non-tier holder paragraph, as the review proposed. That paragraph covers one-slot holders, and the refresh holds the map run's ask: `testMapRun.holdSlot` acquires `mapSlotAsk`.
  - Card 13 lists `internal/gateslot/doc.go`, and its Docs name `lym gate mutate` as a third one-slot holder in the non-tier holder paragraph.
- **F2** (fixed differently): every production function now lands in the card that first calls it from a main.
  - The index card moves after the reach step (old 3 → 6). gatecli first imports `internal/testreach` in card 5, which calls `Open`, `ChangedFuncs`, `Fresh`, `ReadPackageMap` and the reader's methods. Card 5's **Uses** gains `Reader.Maps`.
  - `WriteIndex` moves from card 2 into card 6 with the current, stale-stamp and added-file rows of `TestReaderIndex`. Card 6 also takes the guard's `WriteIndex` clause and pin from card 3. Card 3 can't keep that pin: the pinned call doesn't exist until card 6, and the guard fails on a pinned file that lacks its call.
  - The index read stays in card 2's `Open`, with the no-index and unparsable-index rows. Moving it to card 6 would make card 6 edit `Open` after card 5 uses it, which `validate-plan` refuses as `uses-later-target`.
  - `FuncsReachedBy` moves to card 10 with its `TestReader` row.
  - `ReadReachReport` and `ReachReportMatches` move to card 7 with their `TestGateTestReachStep` rows.
  - `RerunMark.On` moves to card 9 with the `scriptedGate` rerun record and the `On()` subtest. Card 8's wrap still calls `RerunMark.Run`.
  - `gateslot.TriageLines` and `TestTriageLines` move to card 12.
  - Old card 13, the planner, is folded into card 13, `gate-mutate`. It brings `PlanLineMutants`, `fileSites`, `site` and their tests, plus `ChangedLines` and `TestChangedLines` from card 1.
  - The overview's Shared Decisions gain this rule, bounded by `PATTERN-test-only-code`'s rule for a package no main imports.
  - I didn't fold card 8 into 9 or 11 into 12, as the review proposed. Moving the one unreached function of each into its caller's card closes the gap and keeps the cards small.
- **F3**: card 4 lists `internal/loomcli#gateConfigWithTwoTiers` under **Edit:**. Its Intent bullet changes the helper's matched and replacement `code-burler-fix` lines to `reports: [reach, redundancy]`.
- **F4**:
  - Card 4's doc bullet gives the bounds paragraph the three new keys, what each bounds and their below-1 refusal. The doc's refusal list names `reach`.
  - In card 5 and card 13, the gateslot doc's bounds paragraph and `docs/overview.md`'s memory-cap, bounds and memory-guard sentences name each card's run.
- **F5**: card 10's `reachRequest` gets a `limit` field. `reachCmd` sets it to the named 200-commit constant, and `runReach` passes it to `FirstParentCommits`. The range-cap row of `TestGateReach` drives `runReach` with `limit` 2 over the three landings.
- **F6**: card 13 states the outcome past `mutation_budget_min`: no further mutant build or run starts, and every mutant left is reported unjudged with the budget reason and counted. It adds the two budget rows to `TestGateMutate`.
- **F7**: card 2's `Open` reads an absent `sharedDir` or `worktreeDir` as holding no maps, as `PATTERN-lym-files` treats an absent cache. `TestReader` gets the two missing-directory rows.
- **F8**: in card 9, a failure to record the listed functions is now logged at Warn, and the gate returns the inner gate's pass without the turn. The `TestWithReachTurn` row now asserts the pass, no turn and the Warn.
- **F9**: card 7 renders a `Prime` report with the prime reason before any `matchesHead` call. `TestRenderReachReportNote` gets a prime row and an unreadable-report row.
- **F11**: card 13 lists `internal/testcover#site` under **Edit:** and names the new field, `callee`.

## Self-check

- fix-consistency, MEDIUM: the overview's new reach-from-a-main bullet contradicts cards 1 and 2. Fixed as proposed: the bullet now binds only once a main imports the package, and it names the `PATTERN-test-only-code` bound that covers cards 1 and 2.
- fix-consistency, outside its lens (a coverage remark, not a finding): no row tests the prime's "no parent branch" answer in `printReport`'s wiring. Not acted on. The fork itself called it a coverage gap rather than an inconsistency, and the ruling's behaviour is pinned by `TestTriageLookup_Lines`' no-parent-branch row.
- fix-proposal, MEDIUM: under F7, a hub with no maps would make the reach step list every changed function as unreached and trigger a false reach turn. Fixed as proposed: card 5's report judges nothing when `Maps()` is empty, its note names `lym gate test --map`, and a `TestGateTestReachStep` row covers it.
- fix-proposal, LOW: under F6, it was unclear whether the coverage build and planning run past `mutation_budget_min`. Fixed as proposed: card 13 says they run whatever the budget, each build bounded by `redundancy_build_timeout_min`, so the budget row's planned mutants exist.
- fix-claims: no findings.
- fix-affected-set, MEDIUM: the `TestLoadConfig` rows for a tuned `webster` and for a key's lints expect `Reports: [redundancy]` from the template. Fixed as proposed: card 4 says both rows now expect `[reach, redundancy]`.

## Disputed

None.

## Deferred

None.

## Rulings

- **F10**: your parent decision "Triage in the prime follows D7 (plan F10)" (creel #2022) asks that triage in the prime treat the call as having no parent branch. Each mapped failing test then gets the no-change-set line, with no D8 command.
  - Card 11's `mergeBase` now answers "no parent branch" in the prime, never HEAD.
  - The overview's Shared Decisions bullet keeps the prime-from-HEAD rule for the reach step only, and states the D7 behaviour for triage lines.
  - I carried out the whole ruling and sent no flag to the parent.
