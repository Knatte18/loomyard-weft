# Plan round 1 — fixer report

Every finding was checked against the code and the decision record, and every premise held.
All ten are fixed.
`lyx loom validate-plan` reports `ok: true` after the fixes, with informational `create-new-unit` notes only.

## Fixed

- **F1 (test budget).** Confirmed that `cmd/lyx/testdata/test-budget.yaml` rows sit at the current counts (`internal/reedengine` 216, `internal/vscode` 9) and that `internal/segmentcolor` has no row.
  `00-overview.md` gains a Shared Decision, "Test budget": a card that changes a package's top-level test count sets that package's row to its count after the card, and a package new with tests gains a row.
  `cmd/lyx/testdata/test-budget.yaml` is an Edit target of cards 1–11 and 15, and each card's Tests names the rows it sets.
  Card 1 adds the `internal/segmentcolor` row, and card 6 drops the `internal/tokenvocab` row.
- **F2 (RepoName in tmux-tier tests).** Confirmed `RepoName: "test-repo"` at `ensuresession_integration_test.go:53`, `mouse_boot_integration_test.go:57` and `contract_integration_test.go:639,758`.
  Card 6 lists the three files as Edit targets, and its Intent names their `Geometry` literals among the literals it updates.
- **F3 (ideengine.Spawn callers).** Confirmed the single-value `Spawn` calls in both integration files.
  Card 15 lists `spawn_clean_integration_test.go` and `spawn_hub_integration_test.go` as Edit targets and states that they take the two-value return.
  Its ImpactSummary no longer claims one caller.
- **F4 (attach re-assert reach).** Confirmed in `attach.go` that the `window-size` and `#{status}` readbacks return `errAttachChainSuppressed` before `listPanes`.
  In card 3, the pre-flight now marks the window, lists the panes and re-asserts the options right after the strand window resolves, before both readbacks.
  A failed listing skips only the re-assert.
  The attach test now covers the case where either readback suppresses the chain.
- **F5 (card 13 tally).** The ImpactSummary names the four packages and no count.
- **F6 (roles.go).** Confirmed that `MerriamStrandRole` and `RoleRecovery` are declared in `internal/websterengine/roles.go`.
  Card 13 now targets that file, and its Intent says so.
- **F7 (card 3 test economy).** `paneoptions_test.go` keeps only the `strandLabel` table.
  The three color variants become launch fixtures in `spawn_test.go`'s existing `TestLaunchStrandLocked_SplitWindowAndPaneTitleCalls`.
- **F8 (card 8 duplicate assertion).** Confirmed that `reapplyLayout` reaches the apply only through `applyLayoutLockedOpts` with `SkipFocus: true`.
  The `reapply_test.go` assertion and its Edit target are dropped from card 8.
  The `apply_test.go` SkipFocus assertion now also asserts no signal entry.
  Card 9's separate bracket assertion in `reapply_test.go` stays.
- **F9 (VIEW click state).** I took the finding's second option, which adds no new state.
  The `@lyx_strand_window` option is gone.
  The `view` branch now finds the strand window through D3's `@lyx_strands` marker, using a `#{W:#{?@lyx_strands,#{window_id},}}` loop expanded at click time.
  The Bound paragraph now says that this expansion carries only a tmux-issued window id of a window reed marked, and that with no marked window the branch does nothing.
  `paneoptions.go`/`paneoptions_test.go` leave card 7's targets, and the integration test also clicks `VIEW` from a second window.
- **F10 (seed.json start time).** Confirmed that `shedrun.RunDir` places `seed.json` under the durable `_lyx` tree.
  Card 10's Teller and the "Run start time" Shared Decision now state the bound.
  The order holds on the machine where the pair was seeded.
  A rewrite of the file sorts that pair as if its run started then, which can misorder only that pair's revived session.

## Disputed

None.

## Deferred

None.
