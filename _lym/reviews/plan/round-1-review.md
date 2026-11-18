---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/14-gate-mutate.md:44
    summary: Two new gate-slot holders sit outside the places PATTERN-gate-slot allows. lym gate mutate (card 14) holds a non-tier slot, and the reach step's map refresh (card 6) runs inside a plain gate key call, not the --map map run. Neither the gateslot doc's holder list nor pattern/PATTERN-gate-slot.md names either holder, and no card edits them.
    fix: In card 14, add `internal/gateslot/doc.go` under **Edit:**, and extend the Intent's Docs paragraph so that the doc's non-tier holder paragraph (the lines naming the redundancy report step and `lym gate redundant`) names `lym gate mutate` as a third one-slot holder. In card 6, add `internal/gateslot/doc.go` and `pattern/PATTERN-gate-slot.md` under **Edit:**. Its Intent then adds the reach step's refresh, which runs the map run's measurement over a task worktree's anchor maps in the map run's slot ask, to the Gate slot section's list of places in `pattern/PATTERN-gate-slot.md`, beside the map run of `lym gate test --map`, and to the gateslot doc's paragraph on slot holders. Checked with validate-plan --overlay, which reported no new blocking finding.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/03-map-run-writes-index.md:22
    summary: "Several cards create production functions whose only callers land in later cards. Card 3 makes gatecli import internal/testreach, which brings the package into the analysis of the tier1 //lym:guard test TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain. That guard runs at every batch gate, so a batch closing between a creating card and its caller card fails it. The spans: testreach's ChangedFuncs, Open, the Reader methods, Fresh and ReadPackageMap until card 6, FuncsReachedBy until card 10 and ChangedLines until card 14; RerunMark from card 8 to card 9; TriageLines from card 11 to card 12; PlanLineMutants from card 13 to card 14; ReadReachReport and ReachReportMatches from card 6 to cards 7 and 9."
    fix: Make each created production function reachable from a main in the card that creates it. Use `lym loom remove-card` and `lym loom insert-card` to fold card 8 into card 9, card 11 into card 12 and card 13 into card 14, each merged card carrying both cards' targets, Intent and tests. Move card 3's index-write step to directly after card 6, so gatecli first imports testreach in the card that calls `Open`, `ChangedFuncs`, `Fresh` and `ReadPackageMap`. Have card 6 call `ReadReachReport` and `ReachReportMatches` from production code, or move them into card 7, their first caller. `FuncsReachedBy` and `ChangedLines` either move into the cards that first call them (10 and 14), or those cards are placed so that no batch can close between creation and caller. This fix renumbers cards, so it is not overlaid.
  - id: F3
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lym/plan/05-gate-config-reach.md:32
    summary: "Card 5 rewrites the template's code-burler-fix line to reports: [reach, redundancy] but misses internal/loomcli/landingdeps_verify_test.go#gateConfigWithTwoTiers. That helper string-replaces the exact old line to give code-burler-fix tiers [0, 1]. The replace would silently stop matching, so every test built on the helper (parity_test.go, overlay_test.go, recipekeyed_test.go, landingdeps_verify_test.go) reads a code-burler-fix key with tier0 alone, contrary to the helper's doc comment."
    fix: "In card 5, add `internal/loomcli#gateConfigWithTwoTiers` under **Edit:**. Add an Intent bullet saying its matched and replacement `code-burler-fix` lines change to `reports: [reach, redundancy]`, so the helper still declares `tiers: [0, 1]` on that key. Checked with validate-plan --overlay, which reported no new blocking finding."
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/05-gate-config-reach.md:33
    summary: "The cards leave doc text that lists bound keys and bounded runs stale. internal/gateslot/doc.go's bounds paragraph documents every bound key and its below-1 refusal, and names the runs that take the three redundancy_* bounds and run fewer tests side by side. Card 5 adds reach_budget_min, mutation_max_mutants and mutation_budget_min without extending it, and cards 6 and 14 make the reach refresh and lym gate mutate take those bounds without naming them there. docs/overview.md's memory-cap, bounds and memory-guard run lists likewise name only lym gate test (redundancy step included), lym gate redundant and lym gate test --map."
    fix: "In card 5's Intent, add to the `doc.go` bullet that the bounds paragraph of `internal/gateslot/doc.go` gains the three new keys, what each bounds, and that LoadConfig refuses each below 1. In card 6's and card 14's Docs paragraphs, add their run to the same paragraph's sentence naming the runs that take the three `redundancy_*` bounds and run fewer tests side by side. Add their run as well to `docs/overview.md`'s sentences on the memory cap, the build and test bounds and the memory guard; both cards already list `docs/overview.md` under **Edit:**, and the gateslot doc is already card 5's Edit target and joins cards 6 and 14 through F1's fix."
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/10-reach-verb.md:66
    summary: "The 200-commit range-cap refusal of lym gate reach --test has no reaching input. The card says each refusal is reached by a TestGateReach row with its input named, but the fixture holds three landings, and the cap is a fixed named constant of 200 with no injection point on reachRequest or runReach."
    fix: In card 10, give `reachRequest` a `limit` field that `reachCmd` sets to the named 200-commit constant and `runReach` passes to `Fabric.FirstParentCommits`. State in the Intent that the range-cap row of `TestGateReach` drives `runReach` with a `limit` of 2 over the three landings, and asserts the refusal naming the cap and a later `--since`.
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/14-gate-mutate.md:43
    summary: "Two budget branches of lym gate mutate are unspecified or untested. The not-judged branch past reach_budget_min has no TestGateMutate row. The mutation_budget_min bound states no outcome for mutants left when it runs out (unjudged, or not run and counted) and has no row."
    fix: "In card 14, state what happens past `mutation_budget_min`: the check starts no further mutant build or run, and every mutant left is reported unjudged with the budget reason and counted in the unjudged count. Add two `TestGateMutate` rows. Each drives `mutateCmd` with a `minuteUnit` so short that the budget is spent at the hold: one for `reach_budget_min`, where a package with a stale anchor map is listed not judged with its budget line, and one for `mutation_budget_min`, where the planned mutants are reported unjudged."
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/02-testreach-reader.md:35
    summary: "Card 2 never says what Open does with a map directory that does not exist, a common case for a task worktree with no anchor maps or a hub whose --map never ran. Card 11 depends on it. An Open error gives no triage line anywhere, yet card 11's TestGateTest_Scenario row expects the not-in-the-coverage-maps line in a hub where no map holds the test, which passes only if an absent directory reads as empty."
    fix: In card 2's Intent under `Open`, state that an absent `sharedDir` or `worktreeDir` reads as a directory with no maps and refuses nothing, as an absent cache does under `PATTERN-lym-files`. Add a `TestReader` row in which a missing worktree maps directory still answers from the shared maps, and a missing shared directory answers with no reach.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/09-reach-turn.md:68
    summary: "WithReachTurn returns a failure to record the listed functions as the gate's error, which stops a writer's verify gate over an advisory note. ReachPendingOf, by contrast, swallows its read and match failures with a Warn so that the note never stops a gate, as D3 has it."
    fix: "In card 9, change the record-failure bullet: a failure to record the listed functions is logged at Warn with the path and cause, and the gate returns inner's pass without the turn. Change the matching `TestWithReachTurn` row (the listed path held by a directory) to assert the pass, no turn and the Warn."
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/07-reach-review-note.md:53
    summary: "renderReachReportNote gives a prime-written report the wrong reason, and one reason has no test row. A Prime report judges, so matchesHead is called and is always false, and the line says it was written for another tree when D5 wants the true reason. The unreadable-report reason has no TestRenderReachReportNote row."
    fix: In card 7, add a reason, written in the prime with no parent-branch base, that `renderReachReportNote` renders for a report whose `Prime` is set, before calling `matchesHead`. Add `TestRenderReachReportNote` rows for a prime report and for an unreadable report.
  - id: F10
    severity: LOW
    class: decision
    origin: handler
    location: _lym/plan/11-failure-triage.md:40
    summary: "D7 says that a call with no parent branch has no change set, and that each mapped failing test's line then says so, with no D8 command. Card 11 has the prime, which the overview's Shared Decisions say has no parent branch, answer HEAD as its merge base instead. The prime then gets reach and reaches-none lines with a D8 command, and no Added after Discussion entry licenses that departure."
    fix: In card 11, make `mergeBase` answer no parent branch in the prime too, so the prime's mapped failing tests get the no-change-set line naming the missing parent branch and no D8 command, as D7 states. Drop the prime clause from the overview's Shared Decisions bullet as it applies to triage lines. Alternatively, ask the parent to record the prime-from-HEAD choice in the decision record with `lym loom decision add`, and cite it in card 11.
  - id: F11
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/13-line-mutant-planner.md:36
    summary: "Card 13's Intent adds a callee import-path field to each site fileSites returns, but the site struct that gains the field is not among the card's targets; only fileSites is listed under Edit."
    fix: In card 13, add `internal/testcover#site` under **Edit:** below `internal/testcover#fileSites`, and name the new field in the Intent. Checked with validate-plan --overlay, which reported no new blocking finding.
---

### [BLOCKING] New gate-slot holders outside PATTERN-gate-slot's places

**Location:** `_lym/plan/14-gate-mutate.md:44`, `_lym/plan/06-reach-step.md:72`

**Issue:** `pattern/PATTERN-gate-slot.md:12-18` admits a gate build or test in three places only:
a tier run, the map run of `lym gate test --map`, or one of the non-tier holders `internal/gateslot`'s doc lists.
`internal/gateslot/doc.go:62-67` lists two such holders, the redundancy report step and `lym gate redundant`.
Card 14's `lym gate mutate` holds a slot through `mutationCheck.holdSlot` and builds and runs mutant binaries in it.
Card 6's refresh runs `testMapRun` inside a `lym gate test --gate <key>` call, not under `--map`.
The decision record lists `PATTERN-gate-slot` as a constraint, and no card edits the holder list or the background, so both new holders break it.

### [MEDIUM] Production functions unreachable from a main between cards

**Location:** `_lym/plan/03-map-run-writes-index.md:22`, and cards 6, 8, 11 and 13

**Issue:** `cmd/lym/testonlycode_linux_test.go:78-80` is a tier1 `//lym:guard` test.
The narrowed gate's guard step runs it whatever the impacted set, and the `batch` key runs tiers 0 and 1.
`pattern/PATTERN-test-only-code.md:36-38` puts a package no main imports outside the analysis, so cards 1 and 2 are safe.
Card 3's `testreach.WriteIndex` call in gatecli makes the package a main import.
From then on every testreach export without a production caller fails the guard, until cards 6, 10 and 14 call them.
The same gap holds between card 8 and card 9 (`RerunMark`), card 11 and card 12 (`gateslot.TriageLines`), card 13 and card 14 (`PlanLineMutants`), and card 6 and cards 7 and 9 (`ReadReachReport`, `ReachReportMatches`).
`loom-plan-spec.md`'s "Plan vs. schedule" requires a card never to depend on a later card.
Here each such card's batch gate depends on the later card's caller.

### [MEDIUM] Template line change breaks a test helper's string replace

**Location:** `_lym/plan/05-gate-config-reach.md:32`

**Issue:** `internal/loomcli/landingdeps_verify_test.go:27-37` builds `gateConfigWithTwoTiers` by replacing the template's exact text `  code-burler-fix: {tiers: [0], scope: diff, reports: [redundancy]}`.
Card 5 changes that line to list `reach` first.
The replace then matches nothing, and the helper's own doc ("Its `finalize` and `code-burler-fix` keys list both tiers") becomes false for every test that calls it.

### [MEDIUM] Bound-key and bounded-run doc text left stale

**Location:** `_lym/plan/05-gate-config-reach.md:33`, `_lym/plan/06-reach-step.md:131`, `_lym/plan/14-gate-mutate.md:71`

**Issue:** `internal/gateslot/doc.go:174-189` documents each bound key and its refusal below 1.
Its line 179 names the runs that take the three `redundancy_*` bounds: the redundancy report step and `lym gate redundant`.
Card 5 adds three bounded keys and edits only the `reports` paragraph.
Cards 6 and 14 add two bounded runs and update only the template comments, the gatecli doc and the test-speed background.
`docs/overview.md:970,980,995-996` names the runs under the memory cap, the bounds and the memory guard, and no card extends those sentences.
The cards update part of this doc set and leave the rest stale; F1's fix adds the holder list, and this finding covers the remainder.

### [MEDIUM] Range-cap refusal of lym gate reach has no reaching input

**Location:** `_lym/plan/10-reach-verb.md:66`

**Issue:** Card 10 says each refusal is reached by a `TestGateReach` row with its input named (line 59).
The fixture holds three landings (line 71), and the cap is a named constant of 200 with no injection point.
No named row can pass over the cap.

### [MEDIUM] Budget branches of lym gate mutate unspecified or untested

**Location:** `_lym/plan/14-gate-mutate.md:43`

**Issue:** The not-judged reason "past `reach_budget_min`" (line 43) has no `TestGateMutate` row.
The `mutation_budget_min` bound (line 44) states no outcome for the mutants left when it runs out, and no row reaches it.
D11 bounds the verb by both budgets.

### [LOW] Open's behaviour on an absent map directory unstated

**Location:** `_lym/plan/02-testreach-reader.md:35`

**Issue:** Card 2 lists the pass-over rules for maps and for the index, but not for a missing map directory.
Card 11 treats an `open` error as no triage line at all (line 39).
Its `TestGateTest_Scenario` row expects "not in the coverage maps" in a hub where no map holds the test (line 72), which passes only if an absent directory reads as empty.

### [LOW] Reach turn halts the gate on a record failure

**Location:** `_lym/plan/09-reach-turn.md:68`

**Issue:** The note is advisory (D3), and `ReachPendingOf` answers false with a Warn on its own failures so that it never stops a gate (line 74).
A failure to write the listed-functions record still returns a gate error that stops the writer's verify gate.

### [LOW] Prime report rendered with the wrong reason, unreadable reason untested

**Location:** `_lym/plan/07-reach-review-note.md:53`

**Issue:** `ReachReportMatches` never holds for a `Prime` report (card 6, line 94).
Card 7 calls `matchesHead` for every report that judged, so a prime report renders "written for another tree".
D5 asks the line to say why no current report exists.
The listed "unreadable" reason has no row in `TestRenderReachReportNote`.

### [LOW] Triage in the prime departs from D7's no-parent-branch rule

**Location:** `_lym/plan/11-failure-triage.md:40`

**Issue:** D7: "When the call has no parent branch, or its merge base cannot be read, D2 has no change set. Each mapped failing test's line then says no change set exists and why".
The overview's Shared Decisions say the prime has no parent branch.
Card 11's `mergeBase` answers HEAD in the prime, so the prime gets reach lines and D8 commands, which D7 rules out.
D3's prime clause concerns the reach step alone and does not license this for triage.

### [LOW] Card 13 edits `site` without listing it

**Location:** `_lym/plan/13-line-mutant-planner.md:36`

**Issue:** `fileSites` returns `[]site` (`internal/testcover/mutant.go:200`).
Card 13 adds a callee import-path field to each site, which changes the `site` struct, yet only `fileSites` is a target.

## Handler review notes

Card order (focus 1) holds for the four named symbols:
- card 5 creates `gateslot.ReportReach` and the `reach` report before card 6 uses them;
- card 8 creates `WriterOnly` and `RerunMark` before card 9 uses them;
- card 9 creates and wires `verifytree.WithReachTurn` itself.

The order defect found is F2, at the batch level.

Fidelity (focus 2):
- D1 is carried by cards 2, 3 and 4, and D2 by cards 1 and 6.
- D3 is carried by cards 6, 8 and 9, D4 by card 6, and D5 by cards 6 and 7.
- D6 and D8 are carried by card 10, and D7 by cards 11 and 12, with the prime departure in F10.
- D9 is carried by cards 13 and 14, D11 by card 14, including its board note, and D12 by card 5.
- No card carries D10, and the record supports that.
  The added entry "Mutation check order: trial before the mutation step (F6)" says D10 is built only after the trial pays, in a later darn, and moves D12's acceptance of `mutation` to D10's build.
  Card 5 keeps `mutation` an unknown report, as that entry says.
- Each Constraint has a carrying card, with `PATTERN-gate-slot` broken as F1 records.

Attack surface (focus 3): each card states what its change lets through and what bounds it.
- Card 8 lets through only results marked `WriterOnly`, which card 9's turn alone sets.
- Card 10's verb is read-only, with the age of its maps stated.
- Card 14 changes only the anchor maps, and its budgets bound it; F6 covers the missing budget outcome.

Verify lines (focus 6): no card carries a `**Verify:**` line.
Every `Edit` card that names symbols lists member glyphs, and no Intent changes a listed member's signature.
The re-sign rule therefore has nothing to flag.

Test economy (focus 5): each new test function covers a new exported function or a new behaviour.
No existing guard scans `testcover.ReadMap` callers, so card 4's `TestCoverageMapReaderBoundary` duplicates nothing.

## Rejected

- guard-reach F5 (the forked-closure `WriterOnly` bypass in card 8 guards a state no production path produces): D3 and the added entry "Reach turn bypasses Darn-Write's cold fixer (round 2 F1)" require the check on the forked closure and mark it settled, so the card carries a decision rather than a defect.
- guard-reach F4's second half (the `Pending` error branch of `WithReachTurn` is unreachable in production): `WithReachTurn` is a general decorator over a `Pending` seam, and a tier0 row pinning its error contract is legitimate. F8 keeps the record-failure half.
