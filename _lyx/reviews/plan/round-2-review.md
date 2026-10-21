---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/02-gate-tier-config.md
    summary: >-
      In a standalone worktree the lyx config gate step that card 2 sends every refusal to writes the file but then fails on the fabric sync it always runs, and refuses outright when no _lyx directory exists, so the way forward ends in an error envelope.
    fix: >-
      In _lyx/plan/02-gate-tier-config.md add the glyph `internal/configcli#setModule` to the Edit group.
      In the paragraph that makes `selectedLayer` map an empty `--layer` to `LayerWorktree` when the dirs tell no board, add that `setModule` then skips the fabric sync for that write, since a worktree with no board has no weft to sync, and answers ok with the message `edited _lyx/config/<module>.yaml`; inside a hub the worktree-layer write of a per-worktree module keeps its sync.
      In the same card, make the standalone `ErrConfigAbsent` way forward (the one that now names `lyx config gate` in place of `lyx fabric reconcile`) also say that the worktree's `_lyx` directory must exist first and name creating it, because the write refuses `not initialized: _lyx/ directory not found` when it is absent.
      In the Tests paragraph, extend the `TestDispatchLayers` standalone row to pass a sync func that fails the test when called and to assert an ok envelope, and extend the standalone `TestLoadConfig` row to assert the `_lyx` clause.
      Checked with `lyx loom validate-plan --overlay` over card 2 with that one glyph added to its Edit group; it reports no finding the baseline `{}` run lacks.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/12-webster-final-gate.md
    summary: >-
      Card 12 rewrites the webster verify verb to render the final gate call but leaves the verb's own integration tests and their fixture off its Edit list, so the card's webstercli gate fails.
    fix: >-
      In _lyx/plan/12-webster-final-gate.md add to the Edit group the glyphs
      `internal/webstercli#newVerifyFixture`,
      `internal/webstercli#TestVerifyVerb_PassRecordsTheTree`,
      `internal/webstercli#TestVerifyVerb_SecondCallOnTheSameTreeSkips`,
      `internal/webstercli#TestVerifyVerb_FailureExitsOneWithFindings`,
      `internal/webstercli#TestVerifyVerb_DirtyTreeExitsOneNamingPaths` and
      `internal/webstercli#TestVerifyVerb_NoVerifySectionExitsZeroWithWarning`
      (all in internal/webstercli/verify_integration_test.go).
      In the Tests paragraph say that `newVerifyFixture` tells a `GateConfig` reader over a seeded gate.yaml whose `final` tier runs the fixture's command as a shell stand-in, that the plan no longer carries a `## verify:` section, and that the rows drive `runVerify` through the `lyx` built from the tree, first on PATH through `lyxbin.PutFirstOnPath`, not parallel, naming that process-global state.
      The no-section row becomes a row where an unloadable or dangling `final` list is returned as the verb's error naming `lyx config gate`.
      Checked with `lyx loom validate-plan --overlay` over card 12 with those six glyphs added to its Edit group; it reports no finding the baseline `{}` run lacks.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/13-landing-gates.md
    summary: >-
      Card 13 lists the seeded tree_scope_verify values its new load refusal would reject in three places but misses the one inside TestWire_DarnRecipe, which it lists only for the dropped verify key.
    fix: >-
      In the last paragraph of the Tests section of _lyx/plan/13-landing-gates.md add that `TestWire_DarnRecipe`'s subtest `with tree scope on the verify gate's told command stays the hub's whole verify` seeds `tree_scope_verify: "run {targets}"` through `seedDarnHubConfig`, and that the value becomes `lyx gate test --wait --tier 1 {targets}`.
      No glyph changes: `internal/loomcli#TestWire_DarnRecipe` is already in the Edit group.
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/02-gate-tier-config.md
    summary: >-
      Card 2 has the tier_parallel refusal and the undeclared-tier drop warning name lyx config gate --layer machine, a layer that does not apply in a standalone worktree, where the same key sits in the one worktree file.
    fix: >-
      In _lyx/plan/02-gate-tier-config.md change the `tier_parallel` refusal to name the key and `lyx config gate`, leaving the layer to the clause `withConfigWayForward` already appends from the files the load stacked (it names `--layer machine` only when a machine file applies).
      Make the drop warning for an undeclared tier name `lyx config gate --layer machine` only when `configengine.LayerFile` reports the machine layer applies for the load's dirs, and plain `lyx config gate` otherwise.
      Add one standalone row to `TestLoadConfig` asserting a worktree-file `tier_parallel` entry for an undeclared tier drops with a warning that names no machine layer.
---

### [MEDIUM] The standalone lyx config gate way forward ends in an error envelope

**Location:** `_lyx/plan/02-gate-tier-config.md` (the `dirsOf` and `selectedLayer` paragraph and the `TestDispatchLayers` row)

**Issue:** Every standalone refusal in the plan (no worktree `gate.yaml`, no tier, a dangling gate list, a `{args}`-less tier) sends the reader to `lyx config gate`, and card 2 makes that command select `LayerWorktree` when the dirs tell no board.
That path is `setModule`'s worktree-layer fallthrough, which writes the file and then always calls `sync`, the real `fabriccli` `sync` verb.
That verb opens the fabric over the worktree's weft sibling and reads `fabric.yaml`, and a worktree outside every hub has neither.
I ran the deployed `lyx config reed --set width=100` in a scratch standalone repository with a `_lyx` directory: the file was written and the envelope was `ok: false` with `edited _lyx/config/reed.yaml but fabric sync failed: ... fabric.yaml not found; run "lyx config reconcile"`.
Without the `_lyx` directory the same call refuses first with `not initialized: _lyx/ directory not found`, which names no way forward.
So from the state the refusal leaves, the value can be written only after creating `_lyx` by hand, and the write then reports failure.
Card 2's `TestDispatchLayers` row injects a fake sync, so it reads the file back through `gateslot.LoadConfig` and passes without reaching either failure.
A hub is unaffected: an empty `--layer` keeps selecting `LayerDefault`, where the prime writes and commits in `_board`.
The round 1 fix for this path made the layer selection work; the sync and the `_lyx` prerequisite behind it were not covered, and no prior dispute touches them.

### [MEDIUM] Card 12 leaves the verify verb's tests on the retired plan section

**Location:** `_lyx/plan/12-webster-final-gate.md` (Edit group and Tests paragraph)

**Issue:** Card 12 changes `websterCLI.runVerify` to render `GateConfig().GateCall("final", "--all")` and states that its no-section `verified: false` answer goes.
`internal/webstercli/verify_integration_test.go` holds five tests of that verb and the `newVerifyFixture` helper they share.
The fixture writes the command into the plan's `## verify:` section and builds a `websterengine.Geometry` literal with no `GateConfig`.
After card 12 the verb reads neither, so the nil reader is called and the no-section test asserts an answer card 12 removes.
Card 12 edits only `TestGateParity_VerifyGate` in that package.
`webstercli` is a package card 12 targets, so the card gate runs these tests and fails on them.
The geometry the production verb uses is complete (`hubgeom.WebsterGeometry` and `standalonegeom.WebsterGeometry` set the reader); the gap is only the test-side literal and its assertions.

### [LOW] Card 13 misses one seeded tree_scope_verify its new refusal rejects

**Location:** `_lyx/plan/13-landing-gates.md` (last paragraph of Tests)

**Issue:** Card 13 makes `darnengine.LoadConfig` refuse a `tree_scope_verify` that is not one `lyx gate test --wait` call, and names the seeds it moves: darn's `TestLoadConfig` rows, the `scopedVerify` constant and configcli's `darnTreeVerify`.
`internal/loomcli/wiring_darn_test.go:99`, inside `TestWire_DarnRecipe`, seeds `tree_scope_verify: "run {targets}"` through `seedDarnHubConfig` and then calls `c.wire`, which loads the darn config and would now refuse.
The card lists `TestWire_DarnRecipe` only for wiring without a `verify` key, so an implementer following the text would meet the failure only at the gate.

### [LOW] A machine-layer step is named where no machine layer applies

**Location:** `_lyx/plan/02-gate-tier-config.md` (the `tier_parallel` paragraph of `decodeConfig`)

**Issue:** Card 2 has an invalid `tier_parallel` entry refused naming `lyx config gate --layer machine`, and the drop warning for an undeclared tier name that layer's editor session.
`configengine.LayerFile` returns no file for `LayerMachine` unless a board is told, and `selectedLayer` then answers `the machine layer of gate does not apply here`.
In a standalone worktree the key sits in the single worktree file, so a refusal or warning that names `--layer machine` points at a step that cannot run.
`withConfigWayForward` already chooses its wording from the files the load stacked, so the decode message can leave the layer to it.

## Rejected

- Card order across the gate-tier cards: each seam, field or loader refusal is filled in production wiring by the card that first requires it.
  `Env.Round` is required by the verify entry in card 10 and filled in `loomCLI.wire` and `wireDarn` in the same card.
  `Geometry.GateConfig` (card 12) is set by both geometry constructors, and `loomCLI.wire` builds webster's geometry through `hubgeom.WebsterGeometry`, so loom, webstercli and standalone all carry it.
  Landing's `PublishCommand` and `BaseCommand` (card 13) are filled in `landingDeps`.
  `BeginDeps.Gate` and `RecoverDeps.Gate` (card 15) are filled in `webstercli`'s two deps builders.
  The `gate_tiers` field (card 16) and `Env.RefuseRaw` (card 18) are filled in `loomCLI.wire`, and `shuttleengine.Config.RefuseRaw` is set by `Claude.Prepare` itself.
  Every consumer sits after its producer: `--card` and `--base` (card 9) before the round gate (10), `PutFirstOnPath` (10) before card 12, `FailedTier` (4) and `PublishFailure.Tier` (10) before card 13, `FindTierTimeout` (4) before card 14.
- The hub case of the `lyx config gate` steps: `--set 'tiers.tier<N>={...}'` and `--set gates.<name>=[...]` are admitted by card 1, `refuse_raw` and `guard_tier` are plain list and scalar keys the existing `--set` writes, `tier_parallel` is a machine open map under `--layer machine`, and the category classes restrict only the local layer, so a prime write passes the hub-write guard.
- Retired slot, timeout and guard bounds: card 11 keeps the slot and 10-minute timeout for every line that is not an `IsGateCall` line; card 14 follows cards 10, 12 and 13, and the four production callers of `verifytree.Verify` (`lyx webster verify`, webster's final gate, loom's round gate, landing's gate) all render gate calls by then.
  Until card 14 the sites that still run a raw base command run it through `verifytree.Verify` with its slot and 60-minute timeout.
  Guard coverage matches today: `ModuleGuardTests` and webster's `scanGuardTests` both scan the root module only, the replaced guard step is `--tags integration` over the guard packages, and the new guard step runs `guard_tier` there unless the call ran `guard_tier` or a tier covering it whole.
- `Env.RefuseRaw` and the renamed `gate_tiers` field against `PATTERN-recipe-setting-lookup`: the entry governs a recipe row's `*_setting` keys, and the plan follows the existing `RenameTools` and `VerifyTiers` precedent of told values on `Env`.
- `claudeengine` loading `gate.yaml` through `gateslot.LoadConfig` against `PATTERN-config-strictness`: the strict stance is the `gateslot` module's own, and the package still calls `configengine.LoadOrTemplate` for `claude.yaml` alone.
