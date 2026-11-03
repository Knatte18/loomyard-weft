```yaml
lens: code-tests
outcome: reported
```

**Task:** list each behaviour the diff adds or changes and the test that pins it, from the code-tests lens.

**Per behaviour: the test that pins it**

1. **`hubgeom.WebsterGeometry` sets `WorktreeRoot` to `l.WorktreePath()` and `PaneCwd` to the anchor.**
   - Pinned by `TestWebsterGeometry` in `internal/hubgeom/webstergeom_test.go:62-66`.
   - The fixture includes a subpath-anchor row, so a revert fails it.
   - Covered.

2. **`standalonegeom.WebsterGeometry` sets `PaneCwd` to the target.**
   - Pinned by `standalonegeom_test.go:219-223`.
   - Covered.

3. **`webstercli` wires the hub geometry through (`WorktreeRoot` to the worktree, `PaneCwd` to the anchor).**
   - Pinned by `TestWire_ModeHubSelectsHubMode` in `wiring_test.go:126-131`.
   - This duplicates the `hubgeom` assertion at the wiring layer.
   - It is a wiring check, not a new behaviour, so I raise no finding.

4. **#32: begin-batch, record-batch, recover-batch, fix-plan and accept-audit read git at the worktree root when the anchor has no `.git`.**
   - Pinned by the `verbs_test.go` fixture, which now runs at a `backend` anchor with no `.git` link (the commit message names `TestBeginBatchCmd_HappyPath`).
   - Each verb's happy-path test fails with the repository-does-not-exist error if the code reverts to the anchor.
   - Covered.

5. **#31: `path-missing` and `planglyph.Resolve` resolve a `root:` plan against the worktree root.**
   - Pinned by `TestPersistentPreRunE_PlanDirAnchoredAtSubpath` (`verbs_test.go:~1627-1663`).
   - It uses `Root: "backend"` with an Edit target `README.md` that exists under the anchor, and asserts no `path-missing` or `glyph-not-found` finding.
   - Covered for `validate`, which runs through `c.geom.Index.Validate` and `planglyph.ValidateDispatch`.
   - Not covered: the Run-level `index.ValidateDispatch` at `runlevel.go:648`, the begin-batch dispatch re-resolution at `beginbatch.go:480`, `DoneChecks`/`Delta`/`DetectDrift` at `recordbatch.go:933-1067`, and `batcher.DiskSizes(WorktreeRoot)` at `cli.go:292`, `runlevel.go:511` and `rebaseline.go:110`.
   - The begin-batch dispatch path is reached by the begin-batch tests, but their fixture plan has no `root:`, so no test fails if it reverts to the anchor.
   - `TestRecordBatchCmd_Envelope` (`verbs_test.go:~632`) commits its Create target at `fx.CLI.geom.WorktreeRoot`, which is the worktree root. It also fails if `DoneChecks` regresses to the anchor, because the file is no longer found.
   - `DiskSizes` has no root-sensitive test.

6. **`trackedRel`, `suspectBlobs`, `writtenWorktreePaths` and `failBatchInput` now take both roots (relative paths resolve against `PaneCwd`; the within-tree check and git probes use `WorktreeRoot`).**
   - Pinned by none for the split.
   - The only direct test is `runwrites_test.go:150`, which passes `worktree, worktree`.
   - Every engine-level fixture the diff touches sets `PaneCwd` equal to `WorktreeRoot` (`suspect_test.go`, `batchfail_integration_test.go:53`, `reset_*_test.go`, the `*_helper_test.go` fixtures and the rest).
   - If the two arguments were swapped, or either helper used the wrong root, no engine test fails.
   - The webstercli verb tests run split roots, but they exercise none of these paths: the audit is faked, and `seedPendingFinding` sets `PaneCwd` to the worktree root.

7. **`PaneCwd` consumers switched** (transcript-relative write paths, fork-audit workdir, gate cwds, instruction files, `{{.worktree_root}}` and plan-dir spelling, `verifytree.NewPaths`).
   - Pinned by none for the split.
   - Every engine fixture sets `PaneCwd` equal to `WorktreeRoot`, so a regression that reads `WorktreeRoot` instead of `PaneCwd` passes everywhere.
   - Edits of `MerriamBase` and `cli_test.go` do set `PaneCwd` but not `WorktreeRoot`. They are the only places where the two differ.
   - `TestMerriamBase` (`merriambase_test.go:89`) leaves `WorktreeRoot` empty and sets `PaneCwd: worktree`, so the instruction-file read at `PaneCwd` is pinned.
   - The PATTERN directive at `WorktreeRoot` is pinned by `patternGeom.WorktreeRoot = repoRoot` (`merriambase_test.go:181`).
   - `cli_test.go:339` and `cli_test.go:1255` plant a `CLAUDE.md` directory under `PaneCwd`, which pins the same read.

8. **`RepoRoot` removed, and the verify-fixer's `RepoRoot` input reads `WorktreeRoot`** (`verifyfixer.go:199`).
   - Pinned by the `MerriamBase` PATTERN-directive row for the orchestrator path.
   - The `FixVerifySpawnOrAttach` `RepoRoot` feed has no test that fails if it reads `PaneCwd`.
   - Not covered. This is a subset of the missing split-root coverage in item 7.

**Findings**

- **MEDIUM, design (coverage gap)**
  - Location: `internal/websterengine/suspect.go:268` (`trackedRel`), `suspect.go:319` (`suspectBlobs`), `runwrites.go:72` (`writtenWorktreePaths`), `batchfail.go:93`.
  - Summary: the diff's new two-root contract has no test with `PaneCwd` different from `WorktreeRoot`, so swapping or misusing the roots in these helpers fails nothing.
  - Evidence: `runwrites_test.go:150` passes `worktree, worktree`; the other fixtures set `PaneCwd` equal to `WorktreeRoot`.
  - Fix: add a row where `PaneCwd` is a subdirectory of `WorktreeRoot` and a relative path resolves against `PaneCwd` while the returned relative path and git probe use `WorktreeRoot`. Put it in the existing test of each helper, not as new tests.

- **MEDIUM, design (coverage gap)**
  - Location: `internal/websterengine/beginbatch.go:480`, `recordbatch.go:933,947,980,1067`, `runlevel.go:511,648`, `webstercli/cli.go:292`, `webstercli/rebaseline.go:110`.
  - Summary: only `validate` has a `root:` plan test in a non-`.` anchor hub. The dispatch re-resolution, done checks, drift and `DiskSizes` paths could be moved back to the anchor without any test failing.
  - Evidence: the verbs fixture plan comes from `seedValidPlanDir` with no `root:`; the only `root: backend` plan is in `TestPersistentPreRunE_PlanDirAnchoredAtSubpath`.
  - Fix: seed the verbs fixture plan with a `root:` at the anchor, so begin-batch's `ValidateDispatch` and record-batch's done checks pin the worktree-root resolution.

- **LOW, consistency (test economy)**
  - Location: `internal/websterengine/auditledger_test.go:82`, `auditseverity_test.go`, `acceptbatch_test.go:56` and similar one-line `PaneCwd` additions.
  - Summary: fixtures gain `PaneCwd` equal to `WorktreeRoot` without any assertion that depends on it.
  - Evidence: `AcceptPendingAudit` in `TestAcceptPendingAudit_RefusesPathlessFinding` refuses a pathless finding before reaching either root.
  - These are required for correctness where code reads `PaneCwd`, so I report no finding for that.

No duplicated or redundant new tests: the new `path-missing`/`glyph-not-found` loop is an assertion in the existing subpath validate test, and the `seedAnchoredGitLink` helper is removed, not duplicated.

**Tool calls**
1. `grep` for `trackedRel(`, `suspectBlobs(` and `writtenWorktreePaths(` in `internal/websterengine/*_test.go`. My context had only the diff, not whether any test calls these helpers with two different roots.
