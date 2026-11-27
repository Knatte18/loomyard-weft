```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change in the diff, I list everything that calls, reads or asserts it, and whether the diff keeps it correct.

## Per change, the affected set

- **`fabricSync` signature** (new `opts fabricengine.SyncOptions` argument, in `internal/webstercli/sync.go:25`)
  - Production callers: `acceptaudit.go`, `beginbatch.go`, `fixplan.go`, `fixverify.go`, `rebaseline.go`, `recordbatch.go`, `recoverbatch.go`, `restoreplan.go`, `run.go` and `reset.go`. All pass `c.syncOptions`, so they are covered.
    - Every caller except `reset.go` is now unformatted: `c.syncOptions,"…"` has no space after the comma. See F1.
  - Test callers: `cli_test.go:96`, `sync_integration_test.go:157,201` and `verbs_test.go:1857,1862`. All are updated and covered.
  - Docs: none name the signature.
- **`envMasterStrand` renamed to `masterStrandLookup`** (`batchpause.go`)
  - Callers are `wiring.go:219` and `wiring.go:358`, both updated. The test at `batchpause_test.go:241` is renamed and covered.
  - The only remaining mention is `docs/research/speed/tier1-test-revision-2026-11-06/report.md:280`, a dated research report. That is the measurement-report exception, so no finding.
- **`websterCLI.syncOptions` and `websterCLI.strandName` fields** (`cli.go`)
  - Writers: `wireHub` and `wireStandalone` (`wiring.go:218-220`, `wiring.go:357-359`).
  - Readers: every sync, plus `ResetPairCode` at `reset.go:141`, `refuseOrphanedMaster` at `ownership.go:59`, and the `masterStrand` closure.
  - The `reset.go:55` Long text still names `FABRIC_SKIP_PUSH=1`. That is still true, because wiring reads it through `EnvSyncOptions`. Covered.
- **`ResetPairCode` options source** (`reset.go:141`): now `c.syncOptions`. `newResetFixture` sets the field. Covered.
- **The orphaned-Master guard reads `c.strandName`**: `ownership_test.go` sets the field. Covered.
- **`newVerifyFixture` no longer puts `lym` first on PATH**
  - It has seven callers: `verify_integration_test.go:131,145,167,195,216,256` and `verify_parity_integration_test.go:76`.
  - Each sits under a parent that calls `lymbin.PutFirstOnPath`. Covered.
- **`newSkipGitVerbsFixture` (new)**: the `verbs_test.go` file header was rewritten to describe it. See F3.
- **`TestVerifyVerb_*` folded into `TestVerifyVerb`**: `docs/benchmarks/test-redundancy.md:3338-3342` still lists the old names. That file is a generated measurement report, already stale before this diff (it lists `TestVerifyVerb_NoVerifySectionExitsZeroWithWarning`), so no finding.
- **`TestWireLanding_*` folded into `TestWireLanding_FillsTheLandingSeamsPerRecipe`**: no doc, keep-marker or report references either old name. Covered.
- **`pairHubShape` and the `TemplatePairSlug` moves in loomcli**
  - `hubforge.Shapes()` holds the `{".", [TemplatePairSlug]}` shape (`internal/hubforge/template.go:57`), so `CopyHub` accepts it.
  - `ownercheck_integration_test.go:54` spells the shape literally instead of using `pairHubShape`. See F4.
  - The `seedLoomConfig` comment in the same helper file was damaged. See F2.
- **Serial tests must name their process-global state** (PATTERN-test-speed, "Tests run in parallel")
  - Every serial test in the touched files carries such a comment: the `BOARD_SKIP_GIT` tests, `TestBatchVerbs_RefuseUnderAPause`, `TestCommand_GroupGuard_OutsideGitRepo`, `TestVerifyVerb` and `TestGateParity_VerifyGate`.
  - The rest are earlier serial tests that the diff leaves serial: `TestLoomVerbs_StaleStamp…`, `TestEntryLabels_…`, the notifystamp tests and `TestNewPrePushCheckSkip`. Covered.
- **Commit messages and trailers**: no commit in the range carries a `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer, and none is needed for this lens. The directive declares no PATTERN changes.

## Findings

- **F1 — BLOCKING, consistency.** Location: `internal/webstercli/acceptaudit.go:108` and every other changed `fabricSync` call site except `reset.go`.
  - The diff leaves nine production files unformatted: `c.syncOptions,"…"` / `c.syncOptions,fmt.Sprintf(…)` with no space after the comma.
  - The repository's own guard `TestGofmt_ModuleFilesAreFormatted` (`cmd/lym/gofmt_test.go:49-51`, `//lym:only parent-align finalize`) will fail at Parent-Align.
  - The darn-write gate that ran on this tree does not run that guard.
  - Evidence: `gofmt -l internal/webstercli internal/loomcli` lists `acceptaudit.go`, `beginbatch.go`, `fixplan.go`, `fixverify.go`, `rebaseline.go`, `recordbatch.go`, `recoverbatch.go`, `restoreplan.go` and `run.go`. The base version of `acceptaudit.go` is gofmt-clean.
  - Fix: run `lym loom fmt` on each file.
- **F2 — LOW, consistency.** Location: `internal/loomcli/hubfixture_helper_test.go:373` (diff line).
  - The edit dropped the space in the unchanged `seedLoomConfig` doc comment, which now reads "seedLoomConfig creates&lt;anchorPath&gt;/_lym/config/loom.yaml".
- **F3 — NIT, consistency.** Location: `internal/webstercli/verbs_test.go:11-13`.
  - The rewritten header says only the `ErrRunBusy` test keeps the zero `syncOptions`.
  - The `*_FabricSyncFailureWayForward` tests and `TestRecoverBatchCmd_FabricSyncAndReedBootWayForward` also reach `fabricSync` with `newVerbsFixture` and zero options (`verbs_test.go:1878-1936`), so the header no longer describes every test in the file.
- **F4 — NIT, consistency.** Location: `internal/loomcli/ownercheck_integration_test.go:54`.
  - It spells `hubforge.Shape{Anchor: ".", Pairs: []string{slug}}` instead of the package's new `pairHubShape`, which every other moved test in loomcli uses (`hubfixture_helper_test.go:371`).

## Tool calls

1. Grepped the repository for `envMasterStrand`, the old `TestWireLanding_*` / `TestVerifyVerb_*` names and `fabricSync(open`. Reason: I needed stale references in docs and reports outside the diff.
2. Grepped webstercli `doc.go`, `reset.go`, `sync.go` and `cli.go` for the environment and `EnvSyncOptions` wording, and the touched tests for serial comments. Reason: I needed the docs that describe the environment reads, which the diff pieces don't show.
3. Read the serial comments and file headers of the touched loomcli files. Reason: I needed the headers and doc comments outside the diff's hunks.
4. Listed the callers of `newVerifyFixture`, and the tests in touched files that have no `t.Parallel`. Reason: I needed the set affected by moving the PATH setting, and the serial tests the PATTERN comment rule applies to.
