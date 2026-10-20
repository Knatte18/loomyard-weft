# Fixer report — plan round 1

## Changes

- **F1** (BLOCKING), fixed as proposed.
  Card 7's `darnTreeScope` paragraph now returns a `darnengine.LoadConfig` error through `darnengine.WithEditorWayForward` and every other error as is.
  Its `./tools` step in `TestDarnTreeScope_ClassifiesAtEachCall` also asserts `lyx config darn`'s editor.
  Card 8's `darnVerifySource` paragraph says the tree-scope error carries the editor way forward, so Merge-In's and Finalize's "read verify command" error and Publish's seam error name it.
  Card 2's list of the wrapper's callers adds `darnTreeScope`.
- **F2** (MEDIUM), fixed as proposed and then narrowed after the self-check.
  Card 3 picks the clear-first way forward only when the hub file before the save held a tree the staged file holds none of, and a non-empty `tree_scope_verify`.
  Every other load refusal takes the copy-out way forward.
  `TestDispatchHubWideBoard` gains the row for a save that sets `tree_scope_verify` over the template with no tree.
- **F3** (MEDIUM), fixed as proposed.
  The overview's verify line appends `./contracts/specs ./contracts/stencils`.
  The self-check also added `./internal/loomrecipe`.
  `lyx loom validate-plan` reports ok.
- **F4** (LOW), fixed differently.
  The proposed clause was "then re-run the lyx loom command that refused".
  The clause I wrote is "then re-run the lyx command that refused", because `internal/battencli/arm.go:184` reaches `RecipeOverride` from `lyx batten`, where no `lyx loom` command refused.
  The clause is asserted in Card 2's `TestVerifyCommand` row and in Card 7's `TestWire_DarnRecipe` subtest and `TestRecipeOverride` row.
- **F5** (LOW), fixed as proposed.
  Card 2's `tree_scope` template comment and its no-tree refusal both say that a tree path holding `.`, such as `.github`, is written in `lyx config darn`'s editor.
  `TestLoadConfig`'s no-tree row asserts that clause.
- **F6** (LOW), fixed as proposed.
  `TreeScopedCommand` is told a config `LoadConfig` returned.
  It is off only when `TreeScopeVerify` is empty, and it no longer re-trims keys.
  `TestTreeScopedCommand` drops the `tools/` row and the no-tree half of the off row.
- **F7** (LOW), fixed as proposed.
  Card 7 adds `contracts/recipes/darn-recipe.yaml` to `**Edit:**`.
  Its Intent restates the Darn-Burler-Fix gate comment.
- **F8** (LOW), fixed as proposed.
  Card 5 writes the outside guard tests as one `lyx gate test --tags integration <packages> -- -run '^(<names>)$'` step, sorted and deduplicated as in `internal/websterengine/guardstep.go`.
  The overview's second Shared Decision is restated to match.
  `TestCommandFor`'s rows and `TestDerive_OverARepository`'s one-package row expect the single step.
- **F9** (LOW), fixed as proposed.
  The Merge-In `StatusSkipped` step is dropped from Card 7.
  Card 7's `**Uses:**` also drops `internal/verifytree#Verify`, which only that step named.

## Self-check

- fix-claims: no findings.
- fix-consistency, LOW: Card 5's Intent still said the steps "share one prebuilt `lyx`", which contradicts the one-build-per-step sentence and `internal/gatecli/run.go:99-101`.
  Fixed: the Intent now says each step publishes its prebuilt `lyx` under one content-derived store path.
- fix-consistency, LOW: Card 3's new no-tree row omitted the "one commit call" its twin row asserts.
  Fixed as proposed.
- fix-proposal, MEDIUM: F4's "lyx loom command" clause is wrong when `lyx batten` refuses through `RecipeOverride`.
  Fixed as proposed, with "lyx command".
- fix-proposal, MEDIUM: F2's new condition still picked clear-first when the hub held trees and an empty `tree_scope_verify`.
  Fixed as proposed: the condition also requires a non-empty `tree_scope_verify` before the save.
- fix-affected-set, MEDIUM: `perchHardCap` (`internal/loomcli/circling.go:197-202`) reads `darn.yaml` mid-run and returns a malformed-key error raw, so Card 2's widened caller claim was false.
  Fixed differently from the "narrow the claim" option: Card 7 now edits `perchHardCap` to wrap through `WithEditorWayForward` and adds a `TestPerchHardCap` row.
  Card 2's claim now covers every reader of the hub `darn.yaml` that refuses on a load error.
  `darnReviewBudget` falls back on a load error and never refuses, so it is excluded.
- fix-affected-set, LOW: `darn-recipe.yaml` is tested from `internal/loomrecipe`, which the verify did not run, and `darnVerifyGatedRows`' comment in `internal/loomrecipe/darnrecipe.go:19` repeats the stale gate claim.
  Fixed as proposed: Card 7 edits that comment, and the verify line adds `./internal/loomrecipe`.

## Disputed

None.

## Deferred

None.
