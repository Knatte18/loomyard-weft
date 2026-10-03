---
verdict: BLOCKING
rationale: "All ten round-1 findings are fixed in the rewritten plan, but the F4 class (a card deletes a helper whose callers sit in files outside its target list) survives in two cards: card 19 deletes newPortalLauncherTestLocation while its callers in fabricengine/hubworkspace_test.go, launcher_content_test.go and origin_test.go are untargeted, and card 44 deletes reedengine's equalStringSlices while strand_test.go and lifecycle_test.go still call it."
---

# Plan-Review judge, round 1

## Round-1 findings against the rewritten plan

The fixer's rewrite was checked against HEAD (`518dc7400`), finding by finding.

- **F1** holds: card 35 deletes only `TestWire_WebsterRunIsFilled`, `TestWire_PlanSeamsFilled`, `TestWire_ReflectFrictionIsFilled` and `TestWire_SkipPlanReviewIsFilled`, whose bodies at HEAD nil-check `Env` fields and nothing else.
  It trims the four mixed tests to their value and identity checks and keeps `TestWire_LandingSeamFieldsPopulated` and `wiring_commitstatus_test.go`'s `TestWire_CommitStatusFilled` whole.
  Card 36 replaces only `TestWire_SeedChildClosuresFilled`, which at HEAD is five `SeedChild` nil checks.
- **F2** holds: card 42 adds `BuildInto`, and card 43 keeps loomcli's `sync.Once` over an `os.MkdirTemp` directory with error capture.
- **F3** holds: card 2 puts the spawning primitives in the allowlisted `gitkit.go`.
  Card 42 adds the `lyxbin.go` "not governed" row.
  `spawnobservability_test.go` is itself on Tier Purity's `allowedSpawners`, so the `lyxbin.go` path in that row does not trip the new `lyxbin.` token.
- **F4** holds for every card it named.
  A `grep -rlw` of each deleted helper in cards 4, 5, 7, 9, 21, 23, 26–28, 30, 32, 33, 38, 40 and 43 finds no caller file outside its card's target list.
  The class survives in two cards F4 did not name (below).
- **F5**: the importer exemption is bounded by the `lyxbin` import ban, with a fifth plant and a stated "What this lets through".
- **F6**: card 11 describes `CloneHub`'s own one-argument message and makes `TestCloneHub_UnboundWeftNamesTwoArgForm` its owner.
  Card 12 points the dropped CLI pins there.
- **F7**: card 11's ImpactSummary now gives a caller blast radius, destroy's flattening included.
- **F8**: kits and conversions are split into 18/19, 37/38, 39/40 and 42/43.
- **F9**: cards 18, 37, 39 and 42 state their package doc.
- **F10**: cards 20 and 37 carry a `go test -race` `Verify`.

The report has no `## Focus departures` section, so there is nothing to ratify or reject.

## Open findings

### [MEDIUM] Card 19 misses three callers of `newPortalLauncherTestLocation`

Card 19 deletes `newPortalLauncherTestLocation` (`internal/fabricengine/portallauncher_test.go:21`), and its only fabricengine target is `portallauncher_test.go`.
At HEAD the helper is also called from `internal/fabricengine/hubworkspace_test.go:20`, `launcher_content_test.go:192` and `origin_test.go:82`, `:109` and `:128`.
`origin_test.go`'s header comment also names it.
After card 19, those files no longer compile, so the suite is red after the card (contrary to D12), and the derived footprint and dependency graph are wrong.
Its ImpactSummary ("Three untagged test files") is wrong for the same reason.
Card 19 is a product of the F8 split, and F4's caller sweep never covered the old card 18.

Fix: add the three files to card 19's `**Edit:**` list, reword `origin_test.go`'s header comment, and correct the ImpactSummary.

### [MEDIUM] Card 44 misses two callers of reedengine's `equalStringSlices`

Card 44 replaces `equalStringSlices`, defined at `internal/reedengine/reconcile_test.go:15`, with `slices.Equal`, and targets only `reconcile_test.go` in reedengine.
The same package calls it at `internal/reedengine/strand_test.go:1086` and `lifecycle_test.go:328`.
Card 15 edits both files, but for `fakeTmux` and before card 44, so it does not cover the deletion.
After card 44 the package does not compile, and card 44's own acceptance grep only proves the definition is gone, not that every call site moved.

Fix: add `reedengine/strand_test.go` and `reedengine/lifecycle_test.go` to card 44's `**Edit:**` list and correct its ImpactSummary.

## Checked and not raised

- Card 9's file list covers every `fabricFixture`/`newFabricFixture` reference.
  Card 4's file list covers every surveyed query helper.
  Card 14's `Width/Height` and `CollapsedRows/MinFullRows` resets occur only in its own four files.
- Every remaining old-policy citation (`loomcli/parity_test.go`, `validate_test.go`, `loomshed/fixture_test.go`, `loomrecipe/fixture_test.go`, `websterengine/beginbatch_test.go`, `template_test.go`) is assigned to card 7, 33 or 45.
- `gitoracle` reaches git only through `gitexec.RunGit`, so the spawn-observability guard does not see it.
  `gitrepoboundary_test.go` reads `internal/gitrepo` with a non-recursive `os.ReadDir`, so the nested package does not disturb it.
  Card 17's `checkedCallPinnedRawSites` row covers the checked-call guard.
- The test files in the `mergeresolve` and `frictionengine` packages keep their own Run-only `Shuttle` fakes, and no card converts them.
  No import cycle bars `shedfake` from either package, since none of `shedfake`'s four imports depends on them.
  Whether the acceptance criterion's "no consumer package keeps its own ... `Shuttle` ... or merge-shuttle fake" reaches a seam's own defining package is left to round 2 as a focus question, not raised here as a finding.
