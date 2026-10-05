---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/configsync/configsync.go:24
    summary: Rewritten legacyConfigModules comment and extended clone.go reconcile comment still break mid-sentence at a fixed column
  - id: F2
    severity: NIT
    class: consistency
    location: docs/shared-libs/configengine.md:165
    summary: The error-rewrapping note lost its dashes, so the caller list reads as an unpunctuated run-on after Load
  - id: F3
    severity: LOW
    class: scope
    location: internal/fabriccli/fabric.go:785
    summary: Reconcile's non-fatal hub_config_detail path (hub-wide config commit or push failure, exit code unchanged) is documented in the verb help but reached by no test
---

Range: no `_lyx/loom/rework/` directory exists, so the range is `git diff $(git merge-base main HEAD)..HEAD` with `product.parent` = `main`.
The build, `go vet` and the unit tests of the touched packages pass, and `TestBolt_CommitWritten` and `TestConfigHubWideCommitsInBoard` pass under `-tags integration`.
Each focus question is settled below; the three findings are the only departures.

Focus 1, comment line breaks: the comments the fix commits re-broke (`configreg.go`, `bolt.go`, `configcli.go`, `fabric.go` reconcile healing comment, the new `configsync.go` doc comments) follow `PATTERN-comment-line-breaks`.
Two changed comments do not (F1).
A one-word rename inside an otherwise untouched fixed-column comment (`ReconcileHubWideAt` in the `fabric.go` recorder comment, `legacyConfig` in `Result.MigratedFrom`) is not a finding: the rule never rewraps untouched lines.

Focus 2, parallel tests: `TestDispatchHubWideBoard` and its rows call `t.Parallel`, and each row builds its own `newHubFixture`, so no mutable state is shared.
`TestBolt_CommitWritten` calls `t.Parallel` on the parent; its subtests run sequentially by design, because they share one repo whose state each step builds on, so this is not a defect.
`TestConfigHubWideCommitsInBoard` names why it is serial (`t.Setenv`).

Focus 3, `Bolt.CommitWritten`: every contract in the doc comment is reached by a `TestBolt_CommitWritten` subtest (only the written paths, write error unwrapped, deletion staged, empty list, `SkipGit`), and no subtest duplicates another's coverage.

Focus 4, docs and help: `boardengine.LoadConfig` does detect absence with `errors.Is(err, configengine.ErrNotInitialized)` plus an absent-file check and returns `board config <path> not initialized; run "lyx fabric reconcile"`, which matches the note's content, but not its punctuation (F2).
The `reconcile` Long text names the hub-wide config repair, the commit in `_board` and the keys `hub_config`, `hub_config_detail` and `warnings`, all of which `runReconcile` emits.

### [LOW] Rewritten and extended comments keep column-wrapped breaks

**Location:** `internal/configsync/configsync.go:24`, `internal/fabriccli/clone.go:115`
**Issue:** The `legacyConfigModules` doc comment was rewritten in this range, and its second sentence still breaks mid-clause after "which held" and after "single-key top-level documents that fabric.yaml's".
The inline comment above `CommitAnchoredPaths` in `CloneAndWire` was extended with "or retired", and the sentence it extends still breaks at a column after "on the weft primary".
Both are changed comments, so `PATTERN-comment-line-breaks` governs them.
**Fix:** Re-break both comments one sentence per line, with a further break only at an independent-clause boundary.

### [NIT] Error-rewrapping note lost its separators

**Location:** `docs/shared-libs/configengine.md:165`
**Issue:** The note now reads "The strict callers of `Load` `fabricengine.LoadConfig`, `loomengine.LoadConfig` and `batcher.Active` match the substring", with the dashes that set off the list removed.
**Fix:** Restore the dashes around the list of callers.

### [LOW] Reconcile's non-fatal hub config commit failure is untested

**Location:** `internal/fabriccli/fabric.go:785`
**Issue:** `runReconcile` keeps the exit code at 0 when the `_board` commit or push of the hub-wide config fails, and reports the failure under `hub_config_detail`; the verb's Long text promises this.
`TestRunCLI_Reconcile_HealsMissingRepoWideConfig` covers only the success path, and `TestRunCLI_ReconcileBackfillFailureIsNonFatal` asserts `warp_binding_detail` alone, so no test reaches `hub_config_detail`.
This is a behavior the diff adds with no coverage, so a test is justified under `PATTERN-test-economy`.
**Fix:** Add a case to the reconcile-healing test file: delete and commit away the hub-wide configs in `_board`, point `origin` at an unreachable path, run `reconcile`, and assert exit 0, a `hub_config_detail` naming the failed push, and the files back on disk.
