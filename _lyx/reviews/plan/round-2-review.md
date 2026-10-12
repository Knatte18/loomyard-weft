---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/15-configsync-layer-reconcile.md:45
    summary: Only the Ensure walk ever seeds a per-worktree repo default, so a CloneAndWire hub has no loom base layer and card 9's strict hub load fails from card 9 onward, card 21's own integration row included
    fix: "Make a fresh hub carry the per-worktree repo defaults before any card makes a hub load strict on them, and keep the seed inside `hubreconcile.Ensure`, which round 1's F6 fix established for `PATTERN-binary-change-reconcile`. In card 15, have `fabriccli.CloneAndWire` end by calling `hubreconcile.Ensure(hubgeom.ReconcileGeometry(...), hubreconcile.Options{})` on the new hub. Its build stamp is absent, so the walk runs, seeds every absent per-worktree default from the prime and commits it on the board. Do not pass `SeedDefaults: true` from `CloneAndWire`, which would reopen round 1's F6. State that bound in card 15's Intent. Add `internal/hubforge/doc.go` to card 15's `Edit:` and correct its seeding contract, which today says a clone arrives with every module's config materialised in the weft prime. Then close the window between card 9 and card 15. Either insert a card before card 9 with `lyx loom insert-card` that moves the absent-default seed (the `SeedDefaults` option, its `reconcileHubWide` caller and the `CloneAndWire` Ensure call) ahead of card 9, leaving card 15 the sparse-layer reconcile, or keep loom's callers in card 9 on the transitional `configengine.Dirs{Anchor: ...}` geometry and move their switch to `fabricengine.ConfigDirs` into card 15. Card 21's `stepSeedChild_WritesTheRunSettingsIntoThePairsLocalLayer` must then find a repo default in its hubforge hub."
  - id: F2
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/02-configengine-layer-stack.md:85
    summary: Card 2's strict-load refusal for an absent base under the board dir names lyx fabric reconcile, which card 15 makes leave an absent per-worktree default absent, so for loom the named way forward cannot clear the refusal
    fix: "In card 2's Intent, split the absent board-dir base refusal by module kind. A hub-wide module keeps `run \"lyx fabric reconcile\"`, which seeds hub files. A per-worktree module, where only `hubreconcile.Ensure`'s walk seeds the default, names a step that does create the file. One option is `run \"lyx config <module> --layer default --set <key>=<value>\" from the prime`, which card 17's base-layer write scaffolds from the template and commits on Bolt, with the same refused-session clause. The other is to make `hubreconcile.Ensure` treat an absent per-worktree repo default as a stale stamp and name re-running the start verb. Update the `TestLoad_Refusals` row card 2 describes so it pins the per-worktree way forward beside the hub-wide one. If F1's fix has `lyx fabric reconcile` seed defaults instead, state that bound in card 15 and keep this way forward."
  - id: F3
    severity: BLOCKING
    class: design
    location: _lyx/plan/08-burler-models-layered-load.md:56
    summary: Card 8 stacks layer files inside burlerengine and modelspec, against the decision record's PATTERN-config-layers constraint that only configengine reads or stacks layer files, which card 13 writes into PATTERN.md
    fix: "Move the stacking into configengine. Card 2 (or card 8, as a configengine member) adds an exported configengine function that returns the stacked, unexpanded seed-only document. It merges `Layers(dirs, spec)` in stack order through `yamlengine.Merge` under the spec's open maps and `WholeEntries`, does no template fill, and also returns the `[]LayerDoc` so each reader can still validate per file and name that file. Card 8's `burlerengine.LoadConfig` then decodes that one merged document after validating each layer, and `modelspec.LoadRegistry` overlays the one merged document on `builtins()` by whole-entry replacement. Neither package keeps a per-layer merge loop. Card 7's choice of the highest layer file holding `profiles` repeats configengine's open-map rule in batcher, so give that attribution the same configengine home, for example by also returning the layer each top-level unit came from."
  - id: F4
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/13-configreg-local-bounds.md:43
    summary: The loader cards pin the same local-layer keys that card 13's TestModules_LocalLayerRefusals pins, so each of those behaviours is covered twice although the cards say card 13 owns the per-key refusals
    fix: "Make each key's local-layer verdict pinned once. In card 13's `TestModules_LocalLayerRefusals`, drop the rows a loader card already pins through the real load: refused `reed.shell` (card 4), `shuttle.claude_deny_python` (card 5), `burler.lenses` (card 8), `loom.parent_review_wait_min` (card 9), `darn.verify` and `landing.publish_verify` (card 10); passing `models` alias the template lacks (card 8), `loom.review_max_bounces` (card 9), `darn.writer` and `landing.conflict` (card 10). Keep the acceptance keys no loader card covers: `darn.review_max_bounces`, `landing.require_pr_to_base`, `reed.mouse`, `loom.discussion_producer`, `loom.plan_fan` and `shuttle.claude_prompt_cache_ttl_roles.<role>`. Alternatively, have each loader card's wiring row use a key card 13's table does not hold. Either way, reword the loader cards' sentence that card 13 owns the per-key refusals so it names what card 13 still owns."
  - id: F5
    severity: MEDIUM
    class: scope
    location: _lyx/plan/16-sparse-migration.md:39
    summary: Card 16 writes the sparse marker before the worktree commit, but hubreconcile's snapshot records only files present beforehand, so a restored failure leaves the marker on disk and the next walk never strips that worktree, contrary to D6 and to card 16's own test assertion
    fix: "Add `internal/hubreconcile#snapshotConfig` to card 16's `Edit:`. Its Intent then says the snapshot records `configengine.SparseMarker(anchor)` as absent (nil) when the marker is not on disk, so `restoreConfig` removes a marker the failed walk wrote. The `TestEnsure_MergeBeginningBeforeCommitRestoresTheFiles` assertion card 16 already lists then pins it."
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/plan/02-configengine-layer-stack.md:89
    summary: D5 says a filled key is logged with the layer it came from, but card 2 logs only template fills, so a key a sparse worktree file takes from the repo default is never logged with its layer
    fix: "In card 2's Intent, either log each key the merge takes from a layer below the reading layer with that layer's name (`default`, `worktree`), or state that only template fills are logged because a key inherited through a sparse layer is the normal case and `--resolved` (card 18) shows its layer. Make `TestLoad_LayerStack` pin whichever choice is made."
---

### [BLOCKING] Per-worktree repo defaults are never seeded outside the Ensure walk

**Location:** `_lyx/plan/15-configsync-layer-reconcile.md:45`, with `_lyx/plan/09-loom-layered-load.md` and `_lyx/plan/21-seed-child-run-settings.md:46`

**Issue:** Card 15 seeds an absent per-worktree repo default only under `SeedDefaults`.
Only `hubreconcile.reconcileHubWide` passes it; `fabriccli.runReconcile` and `CloneAndWire` pass false.
In a hub, card 15's `ReconcileAll` also never adds to a worktree file: "an absent one stays absent".
After card 15, then, `CloneAndWire` (`internal/fabriccli/clone.go:54` and `:103`) materialises no per-worktree config at all, neither a repo default under `_board` nor a worktree file in the prime.
Every hubforge fixture inherits that, since `internal/hubforge` never calls `hubreconcile` (grep finds no reference).
Card 9 makes loom load strictly with `fabricengine.ConfigDirs(location)`, so in a hub its base layer is the repo default, and strict `Load` errors when that file is absent.
Card 21's integration step reaches this directly.
`TestBattenIntegration_Rows` drives the rows through `shed.Step` over a `hubforge.CopyHub` hub (`internal/battencli/lifecycle_integration_test.go:261`, `:320`), and Worktree-Create's only reconcile is the `Options{Pair: ...}` call (`internal/battencli/wire.go:261`), which reconciles one worktree and never runs the hub-wide step (`internal/hubreconcile/walk.go:143-157`).
So `loomengine.LoadConfig(fabricengine.ConfigDirs(pair), "loom")` meets an absent default and fails, while the card asserts a budget of 2.
The same holds for the loom loads in `internal/loomcli/wiring.go:418` and `internal/webstercli/wiring.go:128` under the hubforge-based loomcli integration tests that do not go through a start verb.
There is also an ordering defect.
Card 9 lands before card 15, the first card to seed any per-worktree default, so between them every hub's strict loom load, `lyx loom start`'s Ensure walk included, has no base layer.
The integration tier of the plan's own `## verify:` runs at those batch boundaries.
`internal/hubforge/doc.go` says a clone "arrives with every registered module's default config already materialized", which card 15 makes false, and no card edits that file.
This follows from round 1's F6 fix, which rightly moved the seed out of `CloneAndWire` and `lyx fabric reconcile` but left no route that seeds a fresh hub before its first start verb.
The fix keeps round 1's constraint by having the clone call `Ensure` rather than seed by itself.

### [BLOCKING] The absent board-dir base refusal names a way forward that cannot clear it for a per-worktree module

**Location:** `_lyx/plan/02-configengine-layer-stack.md:85`, against `_lyx/plan/15-configsync-layer-reconcile.md:45`

**Issue:** Card 2 has strict `Load` name `run "lyx fabric reconcile"` for any absent base layer under the board dir, "which only the hub-wide reconcile seeds".
That holds for a hub-wide module: `runReconcile` heals an absent hub file today (`internal/fabriccli/fabric.go:627-649`).
For a per-worktree module, card 15 explicitly makes `fabriccli.runReconcile` pass `SeedDefaults: false` and "leave an absent default absent".
loom is the strict per-worktree loader (card 9), so loom's refusal names a verb that leaves the named file absent.
That breaks `PATTERN-refusal-way-forward`: the refusal surfaces through loom's verbs, under the "step arming refused" row of `contracts/specs/refusal-spec.md:173`.
Card 2's `TestLoad_Refusals` row pins the wrong text.
Round 1's fixer report raised this under "Note for the next round" and left it open, because the record does not settle whether repairing a default counts as D6 seeding.
F2's second option keeps the repair inside `Ensure`, so it needs no ruling on that.
The report also limits the gap to a default deleted after a walk, but F1 shows a second input that reaches it: any hub built by `CloneAndWire` before its first start verb.
Card 7's retired-key way forward, by contrast, does hold: see the focus verdicts below.

### [BLOCKING] Seed-only modules stack their layers outside configengine

**Location:** `_lyx/plan/08-burler-models-layered-load.md:56` and `:58`, with `_lyx/plan/07-batcher-layered-load.md:75`

**Issue:** The decision record's constraint introduces `PATTERN-config-layers`: "only `internal/configengine` reads or stacks layer files, and only it forms their paths".
Card 13 writes it into `PATTERN.md` as "only `configengine` reads or stacks layer files and forms their paths".
Card 8 reads through `configengine.Layers`, but then has `burlerengine.LoadConfig` merge the layers "in stack order per name".
It also has `modelspec.LoadRegistry` lay the layers over `builtins()` "in stack order by whole-entry replacement".
That is layer stacking in two packages other than configengine, so the plan lands a PATTERN entry its own code breaks.
Card 2's description of `Layers` writes this split into its contract: the seed-only readers "apply their own fallback".
Card 7's retired-key attribution, "the highest file in `configengine.Layers(...)` holding `profiles`", re-implements configengine's open-map carry rule in batcher to pick the winning layer, a smaller instance of the same split.
Under the explore step's rule, a violation of a listed constraint is BLOCKING.
Round 1's F2 found the same split in card 16's configsync, and its fix moved that stacking into `configengine.BelowLayer`.
This is the same gap in other cards, so it is raised once, as a `design` finding about the method: every layer stack belongs in configengine.

### [BLOCKING] The same local-layer key rows appear in the loader cards and card 13

**Location:** `_lyx/plan/13-configreg-local-bounds.md:43`, with the wiring rows of cards 4, 5, 8, 9 and 10

**Issue:** Each loader card adds one local refusal row "proving the loader passes its `ConfigSpec`" and says card 13's `TestModules_LocalLayerRefusals` "owns the per-key refusals".
But each picked a key card 13's table also pins.
On the refused side: `shell` (card 4) and `reed.shell`, `claude_deny_python` (card 5) and `shuttle.claude_deny_python`, `lenses` (card 8) and `burler.lenses`, `parent_review_wait_min` (card 9) and `loom.parent_review_wait_min`, and `verify` / `publish_verify` (card 10) and `darn.verify` / `landing.publish_verify`.
On the passing side: card 8's local alias the template lacks, card 9's local `review_max_bounces` winning, and card 10's local `writer` and `conflict` winning all repeat card 13's pass rows `models` alias, `loom.review_max_bounces`, `darn.writer` and `landing.conflict`.
So the behaviour "a local `<key>` is refused" or "accepted" is pinned twice for each of these keys.
`PATTERN-test-economy` says no two tests cover the same behavior, and the cards' own statement that card 13 owns the per-key verdicts says the same.
The overlap is row-level, not a whole redundant test, because card 13's table also covers keys no loader card reaches.
Round 1's F5 raised this overlap, and its fix trimmed each loader card to one row, but left those rows on keys card 13's table still holds, so the double pinning remains.

### [MEDIUM] A failed walk leaves the sparse marker behind

**Location:** `_lyx/plan/16-sparse-migration.md:39` and `:49`

**Issue:** Card 16 has `ReconcileAll` write the marker after the stripped files and before `hubreconcile.reconcileWorktree` commits.
On a commit that fails, or on a merge that begins before it, the worktree is restored from `snapshotConfig`.
`snapshotConfig` (`internal/hubreconcile/walk.go:349-381`) records only the files present in the config dir plus each registry module's file, so a marker that did not exist before is not in the snapshot.
`restoreConfig` therefore leaves the new marker on disk while it restores the full files.
The next walk then sees the marker and never strips that worktree, which contradicts D6 ("A walk that dies partway ... leaves those worktrees without the marker, and the next walk strips them").
It also contradicts card 16's own assertion that a worktree failed through `beforeCommit` "stays unmarked and the next walk strips it".
Card 16's targets do not include `snapshotConfig`, and card 15's snapshot change covers only local files, so the change the assertion needs has no card.

### [LOW] D5's per-layer fill log is carried only for the template layer

**Location:** `_lyx/plan/02-configengine-layer-stack.md:89`

**Issue:** D5 says "A filled key is logged with the layer it came from", and the strictness constraint says "the missing-key fill runs layer by layer down to the template".
Card 2 logs only `FillMissing`'s template fills, under the layer `template`.
A key a sparse worktree file lacks and the repo default supplies is never logged with its layer.
Either that is the intended reading, and the card should say why, or the card has dropped part of D5.

## Focus verdicts

- Card 2's probe with `Dirs{Anchor, Board}` and the reading worktree's `_lyx/` absent: card 2 runs the probe on `Board` when set and reads an absent `Anchor/_lyx/` as absent upper layers.
  A strict hub-wide `Load` still reads the hub file, and a per-worktree `LoadOrTemplate` still sees the repo default.
  `TestLoad_LayerStack`'s two rows pin both, and cards 10–12 load through the same rule.
  `lyx fabric reconcile` heals the hub-wide files before it loads fabric, so its own loads are not circular.
  Not a defect; the probe's strict per-worktree case is where F1 and F2 arise.
- Card 16's strip: configengine stacks the layers below (`BelowLayer`), and configsync only compares and rewrites the reconciled files, as reconcile does today.
  The strip itself does not break the constraint.
  The constraint is broken in card 8 (F3), with a smaller instance in card 7.
- Card 16's hubreconcile tests: the card only extends `TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree`, `TestEnsure_MidMergePairIsLeftAloneAndRetried` and `TestEnsure_MergeBeginningBeforeCommitRestoresTheFiles`, and creates no new top-level test in `internal/hubreconcile`.
  Not a defect; F5 concerns the code that last assertion needs.
- Card 7's retired-key way forward: batcher's `Migrate` is `batcher.MigrateConfig` (`internal/configreg/configreg.go:66`), which drops retired profile keys.
  Card 15 runs `Migrate` over a present repo default in every `ReconcileHubWideAt` call, `lyx fabric reconcile`'s included with `SeedDefaults` off, and over the sparse worktree and local layers in `ReconcileAll`, which `lyx config reconcile --apply` drives.
  Each layer's named way forward clears a retired key in that layer.
  Not a defect.
- Cards 4–12 against card 13: the overlap is F4.
  No loader card creates a new top-level load test; each extends an existing one.
- Card 15's seeding: the seed is reachable only from `hubreconcile.Ensure`'s walk, and the card states that bound.
  The bound itself is the root of F1 and F2.
