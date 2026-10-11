---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/02-tokencount-run-lower-bound.md
    summary: Card 2 edits the package doc in tools/tokencount/main.go but its Edit list names only the run member, not the file that holds the doc
  - id: F2
    severity: LOW
    class: design
    location: _lyx/plan/04-loom-missing-seed-remedy.md
    summary: Card 4 renders a non-self run-id into the driverless seed command, but its stated bound (a wrong parent is refused at the next bootstrap) holds only for the self run-id
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/05-loom-disagreeing-seed-way-forward.md
    summary: The acceptance criterion that lyx loom step drives a producer after following the disagreeing-seed way forward is not reached; card 5 stops at the reed Up refusal
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/plan/03-loom-help-seeding-routes.md
    summary: docs/overview.md sits under Edit, while the plan spec names docs/overview.md as a Prosa target
---

### [MEDIUM] Card 2's package-doc edit has no target

**Location:** `_lyx/plan/02-tokencount-run-lower-bound.md`, `**Edit:**` and the Intent paragraph "The package doc in `main.go` gains the bound rule".

**Issue:**
The Intent changes the package doc comment at the top of `tools/tokencount/main.go` (above `package main`, line 126), which belongs to no symbol.
The only `main.go` target on card 2 is `tools/tokencount#run`, a member glyph whose span is the `run` function, so the doc edit falls outside every target the card lists.
Card 1 handles the same edit correctly: it lists `tools/tokencount/main.go` itself.

**Fix:**
Replace `tools/tokencount#run` with `tools/tokencount/main.go` in card 2's `**Edit:**` list, as card 1 does.
`redundant-file-target` rules out listing both the file and `#run`, so the file replaces the member.
The plan template allows a file path for content with no symbol to name.

### [LOW] Card 4's wrong-parent bound fails for a non-self run-id

**Location:** `_lyx/plan/04-loom-missing-seed-remedy.md`: "with the addressed run-id rendered into the seed command" and "a wrong value is refused at the next bootstrap's seed stage (card 5)".

**Issue:**
For a run-id other than `self`, card 4 renders `lyx shed seed <run-id> --recipe loom --driver go --param parent=<parent-branch>`.
The bootstrap's seed-agreement check reads and writes only `shedrun.SelfRunID`'s seed (`internal/loomcli/sharedbootstrap.go:100` and `:105`).
So a wrong parent typed into a non-self seed is never compared with the recorded parent, and the card's stated bound does not hold there.
That bound comes from D3, and D3 and the acceptance criteria only name the `self` form of the command.
The non-self rendering is the card's own extension, and its Intent does not say what that extension lets through.

**Fix:**
Use one of two fixes.
The first renders the driverless clause only when the addressed run-id is `self`, and keeps the `lyx loom start` remedy for any other run-id.
The second keeps the rendering and changes the card's bound paragraph to say that for a non-self run-id the typed parent is not checked against the recorded parent.
It also names what then reads that seed's `parent` param.

### [LOW] Card 5 does not reach "after following it, `lyx loom step` drives a producer"

**Location:** `_lyx/plan/05-loom-disagreeing-seed-way-forward.md`, Coverage paragraph.

**Issue:**
Decision-record acceptance criterion 168 ends: "after following it, `lyx loom step` drives a producer".
Card 5's subtest follows the way forward and then asserts only that the second step passes its seed stage and refuses with kind `bootstrap` at reed Up, which the bad-reed fixture forces.
The card says driving a producer needs a tmux server and cites `TestSmokeStepBootstrapWiring`.
But that test drives an agreeing seed that was written directly, not a seed that was cleared and re-seeded through `lyx shed seed`.
So no test reaches this criterion as written.

**Fix:**
Use one of two fixes.
The first adds a wrong-parent case to `TestSmokeStepBootstrapWiring` in `internal/loomcli/smoke_bootstrapwiring_test.go`, the tmux tier that Publish runs, and lists that file under the card's `**Edit:**`.
That case follows the way forward and then asserts that `lyx loom step` dispatches Preflight.
The second states in the card that the criterion is reached only up to the seed stage, so the fixer can defer it explicitly.

### [NIT] `docs/overview.md` typed as Edit

**Location:** `_lyx/plan/03-loom-help-seeding-routes.md`, `**Edit:**`.

**Issue:**
The plan spec's Card granularity section reserves `Prosa` for documentation with no single-symbol owner and names `docs/overview.md` as an example.
Card 3 lists `docs/overview.md` under `**Edit:**` beside its Go targets.

**Fix:**
Move `docs/overview.md` into a `**Prosa:**` group on card 3.
The card keeps its `ImpactSummary` because its `Edit` group remains.

## Focus verdicts

- Cards 3, 4 and 5, refusal reachability: no defect in reachability.
  `genericShedVerb` (`internal/loomcli/arm.go:62`) includes `goto`, so the new `goto` row in `TestResolveRunID` reaches the refusal.
  `TestVerbRefusals`' `Run_SeedMissing` row reaches `loomPreRun`'s status-file refusal, because shedverbs' run body calls the PreRun hook first (`internal/shedverbs/run.go:31`).
  `loomPreStep` runs `seedAndCommitBootstrap` before `reed.Up` (`arm.go:521` before `arm.go:534`), so card 5's wrong-parent step refuses at the seed stage with kind `unseeded` before the fixture's bad reed config is reached.
  The remaining gaps are F2 and F3.
- Cards 1 and 2, member glyphs and re-sign arrows: no defect.
  `CountRun` carries the arrow.
  `line` and `turnPoint` are struct types that only gain fields.
  `workingIntervals` and `countFile` keep their signatures.
  Card 2's missing file target is F1.
- Card 2's bound: the Intent states what the filter can drop and what bounds it, including the open risk.
  No defect.
- Card 3's new test: no existing test asserts the `step` or `run` help.
  The only help test in the package, `TestApproveCmd_HelpNamesAwaitingOrBlocked`, covers `approve`.
  The `test-budget.yaml` row is raised in the same card.
  No defect.
- D1 to D6 fidelity: every decision and constraint is carried by a card.
  The only unlicensed scope found is F2's non-self rendering.
  Card 1 checked `tokencount.sh` and `tokencount.cmd`, and they describe no measurement rule.
- Card 6 typing: `Edit` is correct, since the card also edits `claims_test.go`.
  Card 7: `Edit` on Go members is correct.
  Both ImpactSummaries state a real conclusion.
- `## verify:` covers every targeted package under both tiers, in the order the spec requires.
  The repository has no nested module other than test data.
  No defect.
