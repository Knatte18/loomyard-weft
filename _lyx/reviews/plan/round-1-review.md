---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lyx/plan/23-pattern-shorten.md
    summary: Card 23 shortens PATTERN.md only after cards 3, 7 and 19 grow it, so the per-card lint-pattern step fails each of those three card gates, since PATTERN.md already sits above the headroom line.
    fix: Move card 23 ahead of card 3, its text unchanged. Remove it with lyx loom remove-card 23, then re-insert the same card file content with lyx loom insert-card at the position before gate-call-renders-key. The overview Card Index follows the renumbering those verbs write. Every later PATTERN.md growth (cards 3, 7, 19 and 33) then starts from the shortened overview, and each intermediate size stays at or below the final projection that pattern-headroom already admits.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:affected-set
    location: _lyx/plan/01-gate-key-shape.md
    summary: Retirement cards enumerate the readers of what they change by hand and miss members. Card 1's Gates type change breaks compiling of tier0 tests, and cards 6 and 14 leave dead code and stale text naming retired keys.
    fix: "Card 1, under Edit, add internal/gateslot#templateDefaults, internal/websterengine#TestCardGateCommand, internal/websterengine_test#TestRecoverSpawnOrAttach, internal/websterengine_test#TestBeginBatch_PromptFile, internal/loomshed#TestVerifyGate_Scenario and internal/loomshed#roundGateFixture.round. Its Intent moves each onto map[string]gateslot.GateKey or keyed gates: entries, and moves roundGateFixture.round from landing to finalize until card 4 deletes it. Card 6, under Edit, add internal/darnengine#withTreeScope -> func withTreeScope(trees string) string and docs/overview.md, whose tree_scope_verify sentence changes to the row key over the trees' targets. Under Delete add internal/darnengine#treeScopeOffCommand, internal/darnengine#treeScopeVerifyCommand, internal/darnengine#treeScopeTargetsMarker and internal/darnengine#treeScopeControlOperators. Card 14, under Edit, add internal/gatecli#TestGateTest_Scopes, whose guard_tier subtest is renamed and reworded to the per-tier guard group, and docs/overview.md, whose guard_tier mention goes. Each card's Intent names its readers by a search for the changed field, key or symbol, not from memory."
  - id: F3
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/plan/00-overview.md
    summary: The retag cards 24 to 32 are Custom on a false premise. Edit over the same package self glyphs passes the plan gate, so Custom is not a last resort. The scan-after-last-retag departure from D5 rests on the same premise.
    fix: "On cards 24 to 32, change the **Custom:** label to **Edit:**, keeping the same package self glyphs, and add one **ImpactSummary:** line: Changes only the build-constraint lines of these packages' test files, splitting a shared helper file where another tier uses it; no production code or caller changes. In 00-overview.md, rewrite retag-rules' second sentence: the retag cards are Edit cards over package self glyphs. Then follow D5 (the one-tag scan lands with the last retag card): move card 33's Create, Edit (PATTERN.md and pattern/PATTERN-test-speed.md), Uses, Intent and Verify into card 32. Remove card 33 with lyx loom remove-card 33, and drop the scan-after-last-retag Shared Decision."
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lyx/plan/04-verify-gate-site-key.md
    summary: Cards 4 and 14 change two members' signatures without the re-sign arrow. darnTreeScope's returned func gains the key, and guardRunOf returns every guard run. The arrow's absence hides card 4's test caller, which the caller-uncovered check would otherwise name.
    fix: "On card 4, write the darnTreeScope bullet as internal/loomcli#darnTreeScope -> func darnTreeScope(l *lyxcwd.Location) func(gateKey string) (string, bool, error), and add internal/loomcli#TestDarnTreeScope_ClassifiesAtEachCall under Edit, its call passing the key. On card 14, write the guardRunOf bullet as internal/gatecli#guardRunOf -> func guardRunOf(plan []tierRun) []tierRun."
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/21-darn-describe-entry-gate.md
    summary: TestDarnRouting is told to assert that Describe's entry gate builds false or true, but DarnRouting projects through shedbuild.RoutingOf, which builds no entry gate and takes no Env.
    fix: In card 21's Intent, assert Describe's entry gate answer (false at a Code-Review budget of 0, true above it) in TestNewDarn_Routing only. TestDarnRouting asserts only that a budget of 0 keeps every edge and that the budget is applied to the perch rows.
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/19-recipe-entry-gate.md
    summary: The entry-gate evaluation error says to fix a non-bool value set in the task's run_settings there. After Seed-Child the board's run_settings no longer feed the pair, so the resume fails again.
    fix: In card 19's Intent, change the way forward for a value the resolved view shows in the local layer to lyx config <module> --layer local --set <key>=<bool> in the task worktree, then the resume. Keep lyx config <module> from the prime for a hub or default layer.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/19-recipe-entry-gate.md
    summary: The evaluation error ends with a generic then the run's resume. The Command producer's switch refusal it replaces named the runnable rerun, lyx batten run <slug>, so the claim that the refusal survives the move loses its concrete next step.
    fix: In card 19's Intent, have the evaluation error name the resume command of the recipe it runs under, lyx batten run <slug> for batten and lyx loom resume for loom and darn. Tell it as a recipe-wide value on Env beside SettingKnown, filled by loomCLI.wire, battenCLI.wire and envkit.FullEnv. Have TestBuild_EntryGate's non-bool rows assert it.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/01-gate-key-shape.md
    summary: The list-shape refusal's alternative way forward, a bare lyx config reconcile, is a dry run and writes nothing.
    fix: In card 1's list-shape refusal sentence, replace (or lyx config reconcile) with (or lyx config reconcile --layer default --apply), keeping lyx fabric reconcile from the prime as the first way forward.
  - id: F9
    severity: LOW
    class: design
    origin: lens:way-forward
    location: _lyx/plan/24-retag-cmd-contracts-tools.md
    summary: The Verify line fails on any covers line, so a tier the orch cleared with --set to an empty list, covers [], keeps the card failing after the named way forward.
    fix: "In card 24's Verify line, replace the awk pattern /covers:/ with /covers: *\\[ *[0-9]/, so only a non-empty covers list fails. Say in the Intent that an empty covers list passes."
  - id: F10
    severity: LOW
    class: scope
    origin: handler
    location: _lyx/plan/33-tier-tags-scan.md
    summary: The D13 follow-up upsert names no type label, which a task entry needs. Its body also omits two old-spelling sites the plan keeps, testtiming's defaultRedundancyTags and the tmux isolation guard's tag sets.
    fix: In card 33's Intent, have the upsert carry kind task and the same type label this task's own entry carries. Add to the follow-up body card 13's defaultRedundancyTags and card 16's isolationTags and isolationTagSets beside the sites it already names.
  - id: F11
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/05-landing-row-keys.md
    summary: An empty Deps.GateKey reading as the producer's default key serves only hand-built test Deps. mergeInEntry and finalizeEntry always set it, so production never meets it.
    fix: In card 5's Intent, drop the empty-GateKey default. Have the hand-built Deps in envkit.LandingDeps, newGateScratch and TestVerifyGate_CheckMapsVerifyResult set GateKey to merge-in or finalize.
  - id: F12
    severity: LOW
    class: design
    origin: handler
    location: _lyx/plan/07-green-record.md
    summary: TestGreenRecord pins, on the private greenRecord.covered and tierFingerprint, the changed-command and -count=1 rows that the same card also adds to the public-surface TestGateTest_Scenario.
    fix: In card 7's Intent, drop the changed-command and -- -count=1 rows from TestGreenRecord, leaving them to TestGateTest_Scenario. Keep in TestGreenRecord the rows the scenario does not reach. Those are the ignored parallel field, the guard step's -run filter, the none scoper, the 20-commit pruning and the malformed-file read.
---

# Plan-Review round 1 — gate-runs-once

The plan carries every Decision D1 to D14, each `Added after Discussion` shortening (card 3 for `PATTERN-test-tiers`, card 23 for the other nineteen) and each Constraint.
Each card is licensed by the decision record.
This includes card 23, the D13 board follow-up on card 33 and the `covers` check on card 24's Verify.
The overview's PATTERN changes match the record.
The retag cards' package set equals the set of directories holding a `_test.go` file outside `testdata`: 169 each.
No card edits this task's own board entry.
The baseline `validate-plan --overlay {}` is clean, and the combined overlay of every fix below that changes targets or a Verify line is clean too.

### [BLOCKING] Card 23 lands after the cards that grow PATTERN.md
**Location:** `_lyx/plan/23-pattern-shorten.md`
**Issue:** Webster's card gate runs `lyx loom lint-pattern` as step 3, before the card's commit (`contracts/stencils/webster/webster-body-implementer.md` steps 3 and 4; `internal/websterengine/cardgate.go:16-21`).
Without `--base`, that step compares the working tree's PATTERN.md with HEAD's (`internal/loomcli/lintpattern.go`).
`pattern.Headroom` reports any growth whose result is above `HeadroomBytes` = 16384 × 95 / 100 = 15564 (`internal/pattern/check.go:20-24`, `internal/pattern/headroom.go:11-20`).
PATTERN.md is 15651 bytes today.
Card 3 grows the `PATTERN-test-tiers` line from 134 to 203 bytes.
Card 7 adds a 122-byte line and card 19 a 133-byte one, so the card gates of cards 3, 7 and 19 each fail.
Card 23, which makes the room, runs after all three.
The plan gate's `pattern-headroom` check passes because it measures only the projection after every card, not each card's own gate.

### [BLOCKING] Hand-enumerated affected sets miss readers on cards 1, 6 and 14
**Location:** `_lyx/plan/01-gate-key-shape.md`, `_lyx/plan/06-darn-tree-scope-key.md`, `_lyx/plan/14-guard-step-per-tier.md`
**Issue:** Card 1 changes `gateslot.Config.Gates` from `map[string][]int` to `map[string]GateKey` and says its listed fixtures keep every package compiling.
Four untagged writers of the old shape are missing:
- `templateDefaults` (`internal/gateslot/config_test.go:21-25`)
- `TestCardGateCommand` (`internal/websterengine/cardgate_test.go:50`, listed only on card 3)
- `TestRecoverSpawnOrAttach` (`internal/websterengine/recoverbatch_test.go:1726`)
- `TestBeginBatch_PromptFile` (`internal/websterengine/beginbatch_test.go:606`)

So the `gateslot` and `websterengine` test builds break at card 1.

The integration side has two more gaps:
- `TestVerifyGate_Scenario` seeds a list-shape `gates:` block that card 1's load refuses (`internal/loomshed/verifygate_integration_test.go:171-175`).
- `roundGateFixture.round` reads `TierList("landing")` (line 215).

Both are covered only from card 4 on.
Card 6 leaves several members naming `tree_scope_verify`:
- the `withTreeScope` helper's `verify` parameter (`internal/darnengine/config_test.go:42-44`)
- four constants that become dead once `isSingleWaitingGateCall`, `TreeScopedCommand` and the checks in `normalizeTreeScope` go (`internal/darnengine/config.go:34-45`)
- `docs/overview.md:487`

Card 14 leaves two `guard_tier` mentions: the `TestGateTest_Scopes` subtest asserting the guard step runs through `guard_tier`'s slot (`internal/gatecli/scopes_integration_test.go:140`), and `docs/overview.md:461`.
This is one recurring gap in how the cards find readers, so it is filed once.

### [MEDIUM] Retag cards use Custom where Edit fits
**Location:** `_lyx/plan/00-overview.md` (Shared Decisions `retag-rules` and `scan-after-last-retag`), cards 24 to 32
**Issue:** `retag-rules` justifies `Custom` because a header-only change "has no member to name" and `edit-test-file` refuses a test file under `Edit`.
`edit-test-file` reads only file self glyphs ending in `_test.go`; a package self glyph is not one.
Overlaying card 25 as `**Edit:** internal/fabricengine#` with an `ImpactSummary` passes `validate-plan --overlay`, and so does the combined overlay with cards 24 to 32 converted.
So `Custom` is not the last resort the rubric requires.
`scan-after-last-retag` moves the one-tag scan off the last retag card, against D5's "lands with the last retag card", because "a `Custom` group combines with no other label".
That rationale falls with the label.

### [MEDIUM] Missing re-sign arrows on darnTreeScope and guardRunOf
**Location:** `_lyx/plan/04-verify-gate-site-key.md`, `_lyx/plan/14-guard-step-per-tier.md`
**Issue:** `darnTreeScope` is `func darnTreeScope(l *lyxcwd.Location) func() (string, bool, error)` today.
Card 4 makes it "take the key", which changes its result type.
`guardRunOf` is `func guardRunOf(plan []tierRun) *tierRun`, and card 14 has it "return every guard run".
Neither bullet carries the arrow.
With card 4's arrow added, the plan gate reports `caller-uncovered`: `internal/loomcli/darntreescope_integration_test.go:55` in `TestDarnTreeScope_ClassifiesAtEachCall`, which card 4 does not list.
That test is listed only on card 5, so the tier1 build breaks between cards 4 and 5.

### [MEDIUM] TestDarnRouting cannot assert an entry gate
**Location:** `_lyx/plan/21-darn-describe-entry-gate.md`
**Issue:** `DarnRouting(budgets)` returns `shedbuild.RoutingOf(recipes.DarnRecipe)` with budgets applied (`internal/loomrecipe/darnrecipe.go:72-92`).
`RoutingOf` builds bare `ProducerDef`s holding only name, routing, segment and `MaxBounces`, with no Env (`internal/shedbuild/routing.go:19-34`).
Neither card 19 nor card 21 changes that, so the routing projection holds no entry gate to answer false or true.

### [MEDIUM] Evaluation error points at run_settings after seeding
**Location:** `_lyx/plan/19-recipe-entry-gate.md`
**Issue:** `lyx board upsert --help` defines `run_settings` as the keys "for the pair's local config at Seed-Child".
Once the pair is seeded, editing the entry's `run_settings` leaves the local layer that `Env.Setting` reads unchanged.
The named fix therefore cannot clear the non-bool value, and the resume fails again.

### [LOW] Generic resume in the evaluation error
**Location:** `_lyx/plan/19-recipe-entry-gate.md`, `_lyx/plan/20-batten-switch-entry-gate.md`
**Issue:** The Command producer's switch refusals end with `deps.Rerun`, `lyx batten run <slug>` (`internal/shedadapters/command.go:96,100`, `internal/battencli/wire.go:747`).
Card 19's replacement ends with "the run's resume", which names no command.

### [LOW] Bare config reconcile as a way forward
**Location:** `_lyx/plan/01-gate-key-shape.md`
**Issue:** `lyx config reconcile --help` says the verb is a dry run without `--apply` and reports hub-wide modules rather than reconciling them.
Board-dir files are reconciled only under `--layer default`.
`lyx fabric reconcile` does reconcile hub-wide config, through `configsync.ReconcileHubWideAt` with `Apply: true` (`internal/fabriccli/fabric.go:684-720`), so the first named way forward stands.

### [LOW] Card 24 Verify fails on an empty covers list
**Location:** `_lyx/plan/24-retag-cmd-contracts-tools.md`
**Issue:** `lyx config gate` has `--set` and no unset.
An orch that sets `tiers.tierN.covers=[]` leaves a `covers: []` line in `--resolved`, which `/covers:/` fails on.
The Constraint's intent, that no tier keeps a `covers` entry that drops a tier, is met by an empty list.

### [LOW] D13 follow-up entry underspecified
**Location:** `_lyx/plan/33-tier-tags-scan.md`
**Issue:** `lyx board upsert --help` says "a task needs a type label", and the card names none, which leaves the implementer to look it up.
The follow-up body lists the old spellings in the hub commands, the package loads, testtiming's spawn scan, card 12's guards and the purity guard.
It omits two other sites where the plan keeps them: card 13's `defaultRedundancyTags` (`tier0,tier1,tier2,integration,tmux`) and card 16's `isolationTags`/`isolationTagSets`.

### [LOW] Default GateKey branch only tests reach
**Location:** `_lyx/plan/05-landing-row-keys.md`
**Issue:** The card has `mergeInEntry` and `finalizeEntry` always set `Deps.GateKey`, and the production landing deps reach `NewMergeIn` and `NewFinalize` only through them.
The empty-key default therefore guards a state that only hand-built test Deps produce.

### [LOW] TestGreenRecord duplicates scenario rows
**Location:** `_lyx/plan/07-green-record.md`
**Issue:** `PATTERN-test-economy` pins public-surface behavior no test covers.
Card 7 plans the changed-command and `-- -count=1` carry breaks both as `TestGreenRecord` rows over private helpers and as `TestGateTest_Scenario` rows over `lyx gate test` itself.

## Rejected

- way-forward: card 1 naming `lyx fabric reconcile` as a dead end. Rejected: `runReconcile` calls `configsync.ReconcileHubWideAt` with `Apply: true`, so it does migrate hub-wide config. Only the bare `lyx config reconcile` alternative is kept, as F8.
- guard-reach side note: batten recipe tests hitting card 19's nil `SettingKnown` guard. Rejected: `battenrecipe`'s `testEnv` uses `envkit.FullEnv`, which card 19 fills, and `battencli`'s tests build over `c.env` from `battenCLI.wire`, which card 19 also fills.
- Focus check on Verify lines: neither card 24's nor card 33's Verify names a raw test command. Card 24's awk only matches `go test` as text in the resolved config.
- Focus check on card order: cards 2 and 3 render `--gate` only after card 2 adds it. Cards 11 to 16 land before the first retag card, as D5 orders. Card 20's new `integration`-tagged test lands before card 24 and is retagged by card 28.
- Focus check on test economy for card 33: no existing test enforces one tier tag per test file, so its new test is not redundant.
