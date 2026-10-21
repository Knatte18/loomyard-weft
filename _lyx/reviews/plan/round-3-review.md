---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/15-webster-prompt-gates.md:3
    summary: Shared fixtures and sibling tests that the new gate reader and the no-board sync skip break are missing from the Edit groups of cards 2, 13 and 15
    fix: |-
      Three card-file edits, checked with lyx loom validate-plan --overlay over the card files this review touches (baseline `{}` and the combined run of the F1 and F3 target changes both ok, the only findings the informational create-new-unit notes).
      Card 2, `02-gate-tier-config.md`: add `- internal/configcli#TestEditOne` and `- internal/configcli#TestDispatchSet` under **Edit:** after `- internal/configcli#TestDispatchLayers`, and add to the Tests paragraph: `TestEditOne` and `TestDispatchSet` build their dirs with no board directory, so their sync rows (`valid edit is synced and reported`, `failing sync is reported with its output` and the two `wantSyncs: 1` rows) first create the board directory, the hub case that keeps its fabric sync; the no-board case is `TestDispatchLayers`' new rows.
      Card 15, `15-webster-prompt-gates.md`: add under **Edit:** after `- internal/websterengine_test#testCardGates` the four lines `- internal/websterengine_test#newBeginFixture`, `- internal/websterengine_test#newRecoverFixture`, `- internal/websterengine_test#newRunFixtureOver` and `- internal/webstercli#newVerbsFixture`, and add to the Tests paragraph: `newBeginFixture` and `newRecoverFixture` tell `Deps.Gate` a `gateslot.Config` declaring `tier0` and the `card` and `batch` lists, `newRunFixtureOver` tells `Geom.GateConfig` a reader over a config declaring `tier0` and a `round` list, and `newVerbsFixture` seeds `<hub>/_board/_lyx/config/gate.yaml` the same way, since `hubgeom.WebsterGeometry`'s `GateConfig` (card 12) reads it; a zero config is refused by `TierList`, so every test built on these fixtures must reach the rendered gate calls instead of that refusal.
      Card 13, `13-landing-gates.md`: replace the Edit line `- internal/loomcli#seedDarnHubConfig` with ``- `internal/loomcli#seedDarnHubConfig` -> `func seedDarnHubConfig(t *testing.T, hub, verifyAttempts string, replacements ...string)` ``, add `- internal/loomcli#TestPerchHardCap` and `- internal/loomcli#TestWire_LanguageSkills` under **Edit:**, and add to the Tests paragraph: `seedDarnHubConfig` drops its `verify` parameter and its replacement of `verify: ""`, which once the template's `verify` line is gone matches inside `tree_scope_verify: ""` and would write the caller's command there; its callers pass one argument fewer.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/12-webster-final-gate.md:47
    summary: Card 12 and card 10 verify fixtures run the built lyx in a plain scratch repo, and the Tests text names no commit of the seeded gate.yaml and no exclude of the run records that land in the worktree
    fix: |-
      Text-only edits, no target changes.
      Card 12, `12-webster-final-gate.md`, Tests paragraph: after the sentence that `newVerifyGateRepo` tells a `GateConfig` reader, add: the seeded `gate.yaml` is written to the scratch repo's own `_lyx/config/gate.yaml`, the file the built `lyx` reads there, and committed, and `newVerifyFixture` and `newVerifyGateRepo` write `lyxdirs.DotLyxDirName` followed by a slash into the repo's `.git/info/exclude`, since a standalone `lyx gate test` writes its run record and log under the worktree's `.lyx` gate directory and `verifytree` refuses an untracked path; `TestVerifyVerb_SecondCallOnTheSameTreeSkips` rests on that exclude, and the told `GateConfig` reader reads the same committed file.
      Card 10, `10-round-gate-tiers.md`, Tests paragraph: after the sentence that `TestVerifyGate_RoundScenario` is rewritten over a scratch module seeded with a worktree `gate.yaml`, add the same commit and exclude, since the scenario's later calls (the round call after the base command, the darn form) run on a tree the earlier call's run records would otherwise dirty.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/10-round-gate-tiers.md:67
    summary: Card 10 and card 13 map old-record and fallback tiers from the landing and publish lists, but no seam hands those lists to loomshed or landingshed
    fix: |-
      Card 10, `10-round-gate-tiers.md`: give `RoundGate` a third field, `GateTiers func(gate string) ([]int, error)`, the pair's `Config.TierList`, which `loomCLI.wire` and `loomCLI.wireDarn` fill beside `Tiers` and `Render`, and have `NewVerifyGate` pass `round.GateTiers` as `publishFailureSteps`'s `tiers` argument.
      Replace the Edit line `- internal/loomshed#checkedPublishFailure` with ``- `internal/loomshed#checkedPublishFailure` -> `func checkedPublishFailure(paths verifytree.Paths, worktreeRoot string, tiers func(gate string) ([]int, error)) (verifytree.PublishFailure, bool)` ``, the line `- internal/loomshed#PublishFailureNote` with ``- `internal/loomshed#PublishFailureNote` -> `func PublishFailureNote(worktreeRoot, verifyDir string, tiers func(gate string) ([]int, error)) string` ``, and add `- internal/loomshed#TestCheckedPublishFailure_MergeCommit` under **Edit:** after `- internal/loomshed#TestPublishFailure` (the plan gate's `caller-uncovered` finds its call).
      State in the Intent that the old-record and off-list mapping read `tiers`, that a `tiers` error ignores the record with a logged warning, as an unreadable record already is, and that `PublishFailureNote` answers `none` then; and that the closure in `loomcli/wiring.go` that calls `PublishFailureNote` passes a lookup built at call time from `gateslot.LoadConfig` over the pair's config dirs.
      Card 13, `13-landing-gates.md`: state in the Intent that `landingshed.Deps` gains `GateTiers func(gate string) ([]int, error)` beside `PublishCommand`, read at each call, that `landingDeps` fills it from `Config.TierList`, that `envkit`'s `LandingDeps` and `nilLegal` follow, and that `verifyGate.recordFailure` reads it for the fallback tier; `Deps`, `landingDeps`, `envkit#LandingDeps` and `envkit#nilLegal` are already in the Edit group.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/02-gate-tier-config.md:47
    summary: Card 2 drop warning for an undeclared tier names the machine editor whenever the machine layer applies, though the entry may sit in the hub file, where that editor cannot remove it
    fix: |-
      Text-only edit in `02-gate-tier-config.md`, in the paragraph that drops a `tier_parallel` entry for an undeclared tier.
      Word the warning, when the machine layer applies for the load's dirs, to name both steps, plain `lyx config gate` for an entry in the hub file and the operator's `lyx config gate --layer machine` editor session for an entry in the machine file, since the stacked document cannot tell which layer holds it; when the machine layer does not apply, it names plain `lyx config gate` alone, as now.
      In the Tests paragraph, make the warning-capturing subtest assert both steps for the machine-applies row and only the plain step, with no `--layer machine`, for the standalone row.
---

### [BLOCKING] Shared fixtures and sibling tests that the cards' new behavior breaks are missing from the Edit groups

**Location:** `_lyx/plan/15-webster-prompt-gates.md:3`, with the same gap in `_lyx/plan/02-gate-tier-config.md:11` and `_lyx/plan/13-landing-gates.md:3`.

**Issue:**
The cards list the tests they rewrite by name, but a changed required reader or a changed skip rule also reaches tests through shared fixtures and sibling tests, and those are unlisted.
The gap recurs in three cards, so it is one finding about the method: enumerate by grepping every caller of each changed constructor, literal and helper, not by the tests the Intent names.

Card 2 makes `setModule` and `editOne` skip the fabric sync for a worktree-layer write when the dirs tell no board (`02-gate-tier-config.md:52`).
`internal/configcli/configcli_test.go` `TestEditOne` builds its dirs with `worktreeDirs(baseDir)`, whose board is empty, and its rows `valid edit is synced and reported` and `failing sync is reported with its output` assert `wantSynced: true` with the outputs `edited and synced` and `fabric sync failed`.
`TestDispatchSet` runs `dispatch` over `makeLayoutAt(baseDir)`, whose hub path is the temp directory's parent and holds no `_board`, so `fabricengine.ConfigDirs` tells no board there either, and two rows assert `wantSyncs: 1`.
Both tests fail under the card's rule, and neither is in the Edit group or the Tests text.

Card 15 adds `Gate` to `BeginDeps` and `RecoverDeps`, and `Run` renders the verify-fix prompt's `GateCall("round")` from `Geom.GateConfig` at its start (`internal/websterengine/runlevel.go`, the `RenderVerifyFixPrompt` call).
`newBeginFixture` (`internal/websterengine/beginbatch_test.go`), `newRecoverFixture` (`recoverbatch_test.go`) and `newRunFixtureOver` (`runlevel_test.go`) build those deps with no `Gate` and no `GateConfig`.
In webstercli, `newVerbsFixture` (`verbs_test.go`) builds `hubgeom.WebsterGeometry(layout)` over a hub that holds stencils and no `gate.yaml`.
A zero `gateslot.Config` has no `card`, `batch` or `round` list and no `tier0`, so `TierList` refuses it (card 2), and every begin-batch, recover-batch and run test built on these fixtures meets that refusal.
Card 15's Tests text names only `TestBeginBatch_PromptFile`'s new dangling-list row and the prompt tests, and none of the four fixtures is in its Edit group.

Card 13 lists `seedDarnHubConfig` but says nothing about what changes in it.
The helper in `internal/loomcli/wiring_darn_test.go` does `strings.Replace(darnengine.ConfigTemplate(), `verify: ""`, "verify: "+verify, 1)`.
Once the card removes the template's `verify` line, the first match of that text is inside `tree_scope_verify: ""`, so each caller passing `"go test ./..."` writes `tree_scope_verify: "go test ./..."` over an empty `tree_scope`, which the darn load refuses.
Callers outside the card's Edit group are `TestPerchHardCap` (`internal/loomcli/circling_test.go`) and `TestWire_LanguageSkills` (`internal/loomcli/wiring_test.go`).
Dropping the parameter is a re-sign, and the plan gate's `caller-uncovered` check requires every caller's member glyph on the card; the fix lists both, and `TestWire_LanguageSkills` stays in card 16's group too.

### [MEDIUM] Card 12 and card 10 verify fixtures run the built lyx in a plain scratch repo without committing the seeded gate.yaml or excluding the run records

**Location:** `_lyx/plan/12-webster-final-gate.md:47`, with the same gap in `_lyx/plan/10-round-gate-tiers.md:77`.

**Issue:**
The verify-verb, verify-gate and round-gate tests now run the rendered `lyx gate test` through the `lyx` built from the tree, with its cwd in a standalone scratch repo.
Outside every hub that child reads the worktree's own `_lyx/config/gate.yaml`, not the reader the fixture tells (D2, card 8), and writes each tier's record and log under `<worktree>/.lyx/gate` (D11, card 4: `WaitDir(worktreeRoot)`).
`verifytree.DirtyPaths` is `git status --porcelain -z`, which skips only ignored files, and `newVerifyFixture` and `newVerifyGateRepo` are bare `git init` repos with one base commit and no exclude.
So a seeded `gate.yaml` the fixture does not commit makes the first call return `StatusDirty` before any command runs.
With it committed, the first call's run records are untracked under `.lyx/`, and `TestVerifyVerb_SecondCallOnTheSameTreeSkips` finds the tree dirty on its second call instead of skipping; the same holds for the rerun path of the webster gate and for the round scenario's second call (a base-command run, then the round call with `--base`).
`newVerifyFixture`'s own comment promises that nothing a verify run writes lands inside the repo, which no longer holds, and the cards' Tests text states neither the commit nor the exclude.
The `.lyx` name must go through `lyxdirs.DotLyxDirName` in the fixture (`PATTERN-lyxdirs-single-declarer`).

### [MEDIUM] Old-record and fallback tier mapping reads the landing and publish lists, but no seam hands them to loomshed or landingshed

**Location:** `_lyx/plan/10-round-gate-tiers.md:67`, with the landingshed half at `_lyx/plan/13-landing-gates.md:77`.

**Issue:**
Card 10 has `checkedPublishFailure` map an old record, or a new one whose `tier` is off its gate's list, to the highest tier of that gate's list, and has `PublishFailureNote` name the failing gate and tier.
Card 13 has `verifyGate.recordFailure` write the highest tier of the failing gate's list when the log holds no failing summary line.
Neither function can reach a list as the plan stands.
`RoundGate` carries `Tiers []int`, the round list, and a `Render` function (`10-round-gate-tiers.md:56`), and `NewVerifyGate`'s only config seam is `round`.
`checkedPublishFailure(paths, worktreeRoot)` and `PublishFailureNote(worktreeRoot, verifyDir)` keep their signatures, though `loomcli/wiring.go` calls the second.
`landingshed.Deps` gains only `PublishCommand`, `BaseCommand` and `PublishSkip` (`13-landing-gates.md:72`), and `verifyGate` is built from `Deps` alone.
Only `publishFailureSteps` gets a `tiers func(gate string) ([]int, error)` parameter, with no stated source for the argument.
Re-signing `checkedPublishFailure` as the fix does also needs `TestCheckedPublishFailure_MergeCommit` on the card, because `caller-uncovered` finds its call in `publishfailure_integration_test.go`.

### [LOW] Card 2's drop warning for an undeclared tier names the machine editor whenever the machine layer applies, though the entry may sit in the hub file

**Location:** `_lyx/plan/02-gate-tier-config.md:47`

**Issue:**
`tier_parallel` is an open map classified `CategoryMachine`, and the template also declares it in the hub file, so `lyx config gate --set tier_parallel.tier5=2` writes a hub-file entry.
`decodeConfig` sees the stacked document and cannot tell which layer holds a dropped entry, yet the card words the warning to name the `lyx config gate --layer machine` editor whenever `LayerFile` reports the machine layer applies.
When the entry sits in the hub file, that editor opens the machine file and cannot remove it.
D1 licenses the machine-file case only, so the card's wording is wider than the decision.
In a standalone worktree the card's wording is correct, since no machine layer applies and the warning names plain `lyx config gate`.

## Rejected

- Standalone `lyx config gate` write, focus item 1: traced to an ok answer. `dirsOf` taking `fabricengine.ConfigDirs(l).Board` gives no board for a standalone worktree, `selectedLayer` maps the empty layer to `LayerWorktree`, `LayerFile` applies it when the board is empty, `configengine.Set` takes the base-layer full write, the `_lyx` clause in the absent-file way forward matches `FindBaseDir`'s refusal, and `setChecked`'s loader check runs `gateslot.LoadConfig` over the standalone stack. The no-board sync skip leaves nothing for a real sync path to reach, so `TestDispatchLayers`' fake that asserts the sync is never called is enough, and `TestConfigOverRealHub` keeps the real sync for the hub case. The only defect on this path is the sibling tests in F1.
- `tier_parallel` refusal and drop-warning text, focus item 2: the refusal leaves the layer to `withConfigWayForward`, which names `--layer machine` only when `ConfigFiles` stacked a machine file, and no machine file stacks in a standalone worktree; the card's template comment and `TierTimeoutLine` name steps that run there. Not a defect for a standalone worktree; the hub-file wording gap is F4.
- Webster verify verb tests and their fixture, focus item 3: card 12 lists all four `TestVerifyVerb_*` rows, `newVerifyFixture`, `TestGateParity_VerifyGate` and the deleted no-section test; the remaining gap there is the fixture's commit and exclude, F2.
- Seeded `tree_scope_verify` values, focus item 3: darn's `TestLoadConfig` rows, `TestDarnTreeScope_ClassifiesAtEachCall`, `TestWire_DarnRecipe`'s tree-scope subtest and configcli's `TestDispatchHubWideBoard` are all listed and reworded; configcli's `TestDispatchLayers` seeds only an empty `tree_scope_verify`, which the new refusal admits. The remaining gap is the shared `seedDarnHubConfig` helper, F1.
