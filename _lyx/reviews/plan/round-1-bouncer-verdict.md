---
verdict: BLOCKING
rationale: "The fixed plan resolves all thirteen round-1 findings, but internal/gitrepo/noforceadd_test.go, a source scan enforcing the Never Force-Add Invariant, is in no conversion card (D2), and card 15 removes loomRowEngines while production internal/loomrecipe/names.go:18 still names it"
---

# Round 1 judgment

## The round-1 report

Every finding in `round-1-review.md` is legitimate under the rubric.
None restates a mechanical check, and none flags a missing `ImpactSummary` on a `Rename` card or an incomplete dependency list.
Each finding is a fidelity gap against the decision record, a target list that misses a file the Intent needs, or an unbounded removal (the attack-surface item).
The report has no `## Focus departures` section, so there are no departures to ratify or reject.

I checked the fixed plan against each finding:

- F1: card 31 creates `internal/gitkit/testmain_test.go` and drops `//go:build integration` from `internal/gitrepo/testmain_test.go`.
  I listed every package with an `integration`- or `smoke`-tagged test file and the build line of each `TestMain` file.
  Cards 30–32 give each of the 40 packages, and the new `tmuxkit` package, a `TestMain` under every tag set that compiles its tests.
- F2: card 12 names `TestCommand_ReviewSubtree_ExactChildren`, `TestRunCLIIn_BareListingNeedsNoGitRepository` and `TestCLIUnknownSubcommand`.
- F3: card 22 targets `smoke_parentreview_test.go` and `bouncer_fixture_test.go`, and carries a smoke `**Verify:**` line.
  Card 21's smoke sentence is now accurate: `burlerengine/smoke_round_test.go` and `treadleengine/smoke_judge_test.go` both call `stencilstore.Reconcile(…, stencils.Registry(), …)`.
- F4/F13: card 28 names the after-commit callback and pins `batten:` in a kept `battencli` test.
- F5: card 15 targets `interruptpolicy_meta_test.go` and `recipe_test.go`, both readers of `loomRowEngines`.
- F6–F12: carried as the fixer report states.
  F9's premise holds: no `_test.go` file reads `"TMUX"` or `"TMUX_PANE"`.

All thirteen are resolved.

## New findings

### [MEDIUM] `noforceadd_test.go` is an invariant scan no card converts

`internal/gitrepo/noforceadd_test.go` machine-checks `CONSTRAINTS.md`'s Never Force-Add Invariant.
It walks `internal/gitrepo`'s own non-test `.go` files (`os.Getwd` then `os.ReadDir`), bans `noForceAddBannedTokens`, and has a floor (`noForceAddMinScannedFiles`).
D2 binds "every test that enforces a `CONSTRAINTS.md` invariant by walking source", and the acceptance criteria repeat it.
No card targets the file.
Card 10 converts `fabricengine/mergevocab_test.go`, which reads its package directory with `os.ReadDir(".")` in the same way.
The round-1 sweep searched for `*_enforcement_test.go` files, `go env GOMOD` execs and `runtime.Caller` root lookups, so it missed this file.

Fix: add `internal/gitrepo/noforceadd_test.go` to card 10.
Convert it to a shallow `scankit` walk of its own package directory over production files.
Keep its token set and floor verbatim, prove it by one planted token, and name the Never Force-Add guard in the Intent.

### [LOW] Removed names stay referenced from files outside the removing card

- Card 15 folds `loomRowEngines` into the one table, so the variable disappears.
  The doc comment of production file `internal/loomrecipe/names.go:16-18` contrasts `RecipeEngines` with "a second hand-maintained table alongside `loomRowEngines`".
  That comment goes stale, and `names.go` is not a target of card 15.
- Card 11 deletes `cmd/lyx/drift_test.go` and `cmd/lyx/longlist_test.go`.
  Two comments still name them: `cmd/lyx/seamsignature_test.go:7` says "`cmd/lyx/drift_test.go` asserts only…", and `cmd/lyx/sandbox_coverage_test.go:39` says "mirrors `longlist_test.go`'s skip pattern".
  Card 11 targets neither file.

Fix: add `internal/loomrecipe/names.go` to card 15 as a comment-only edit naming the one table, and adjust its ImpactSummary to match.
Add `cmd/lyx/seamsignature_test.go` and `cmd/lyx/sandbox_coverage_test.go` to card 11 so their comments point at the CLI walk.

## Other checks with no finding

- Every `*_enforcement_test.go` file in the tree is a target of cards 6–10.
- Every `go env GOMOD` file is a target of cards 3–5.
- `contracts/stencils/registry_test.go` reads its package directory but syncs the stencil registry rather than enforcing a `CONSTRAINTS.md` invariant, so D2 does not reach it.
- `allowedNonHermetic` carries no `go env` entry today.
  Card 5's generic stale removal covers D20's Hermetic Env line.
- No card is `Custom`.
- The plan-level `## verify:` runs the untagged and `integration` tiers over `./...` and vets `smoke`.
  Cards 22 and 29, the two that change smoke files, each run the `loomcli` smoke tier.
