# Fixer report — plan, round 2

## Changes

- **F1** (MEDIUM, design), `_lyx/plan/02-gate-tier-config.md`: fixed, extended past the proposal.
  The Edit group gains `internal/configcli#setModule` and also `internal/configcli#editOne`, since `editOne` runs the same unconditional sync (`internal/configcli/configcli.go:164`) on the editor path the standalone refusals' `lyx config gate` opens.
  The `selectedLayer` paragraph now says both skip the fabric sync for a worktree-layer write when the dirs tell no board and answer ok with `edited _lyx/config/<module>.yaml`, while a per-worktree module's write inside a hub keeps its sync.
  The standalone `ErrConfigAbsent` way forward now says the worktree's `_lyx` directory must exist first and names creating it.
  The proposal's "sync func that fails the test when called" became an assertion that the sync is never called, the form `TestDispatchLayers`' existing `wantSyncs` field takes, and an editor row sits beside the `--set` row.
  The standalone `TestLoadConfig` row asserts the `_lyx` clause.
- **F2** (MEDIUM, scope), `_lyx/plan/12-webster-final-gate.md`: fixed, partly differently.
  The Edit group gains `newVerifyFixture` and the four tests that keep their meaning, and the Tests paragraph states the fixture's seeded `gate.yaml`, the dropped `## verify:` section, and the `lyxbin.PutFirstOnPath` step that makes the rows non-parallel.
  The proposal put `TestVerifyVerb_NoVerifySectionExitsZeroWithWarning` in the Edit group and turned it into the refusal row.
  I did not do that, because the name would then describe the answer card 12 removes.
  The test goes under a new **Delete:** group, and a new **Create:** test, `TestVerifyVerb_GateConfigRefusalIsTheError`, covers an unloadable config and a dangling `final` list, each returned as the verb's error naming `lyx config gate`.
- **F3** (LOW, consistency), `_lyx/plan/13-landing-gates.md`: fixed as proposed.
  The Tests section names `TestWire_DarnRecipe`'s tree-scope subtest seed `run {targets}`, which becomes `lyx gate test --wait --tier 1 {targets}`.
- **F4** (LOW, design), `_lyx/plan/02-gate-tier-config.md`: fixed as proposed.
  The `tier_parallel` refusal names the key and `lyx config gate`, and leaves the layer to the clause `withConfigWayForward` appends.
  `LoadConfig` tells `decodeConfig` whether the machine layer applies, so the drop warning names `--layer machine` only then.
  The standalone row asserting a warning with no machine layer joins the logger-capturing subtest.

`lyx loom validate-plan` reports `ok: true`, with no finding on cards 2, 12 or 13, both after the round's fixes and after the self-check edits.

## Self-check

- fix-claims, LOW: card 2 said the clause names `--layer machine` "only when a machine file applies", but `ConfigFiles` stacks a machine file only when it exists on disk (`internal/gateslot/config.go:56-58`). Fixed as proposed: the text now says "only when the load stacked a machine file".
- fix-consistency, MEDIUM: `internal/configcli/doc.go` still says a worktree write syncs fabric. Fixed as proposed: `internal/configcli/doc.go` joins card 2's Edit group, and the card says the doc states that a worktree-layer write with no board runs no sync.
- fix-consistency, LOW: card 2's ImpactSummary left out the sync skip. Fixed as proposed: the summary now names the ok answer with no sync for every worktree-layer write with no board.
- fix-consistency, LOW: card 2's template comment on `tier_parallel` named only the machine-layer form. Fixed as proposed: the comment now also says a standalone worktree sets the key in its own file with plain `lyx config gate`.
- fix-affected-set, MEDIUM: darn `TestLoadConfig`'s `tree_scope_verify with no tree` and `tree_scope_verify without targets` refusal rows (`internal/darnengine/config_test.go:80,85`) seed values without `--wait`. Fixed with my own change, since the fork left the choice open: card 13 now has those rows seed `--wait` calls, so each still meets the refusal it names whatever order the checks run in.
- fix-proposal, MEDIUM: the sync skip is keyed on an empty board, so a hub whose `_board` is absent or not statable (`internal/fabricengine/junctionnames.go:122-133`) would also skip. Fixed with the fork's second way out: card 2 now states that case plainly. I did not take the first way out, keying the skip on being outside every hub, because `lyxcwd.Location.HubPath` is always set (`internal/lyxcwd/lyxcwd.go:171`), so the board dir is the only hub signal configcli holds; it is also the condition under which the card already sends a hub-wide write to the worktree file.

Every fork returned a report.

## Disputed

None.

## Deferred

None.
