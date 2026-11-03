---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/websterengine/suspect.go:268
    summary: No test runs the new two-root helpers with PaneCwd different from WorktreeRoot, so swapping or misusing the roots in trackedRel, suspectBlobs, writtenWorktreePaths or failBatch fails nothing
    fix: Add one split-root row to the existing test of each helper (TestLoadRunWrites for writtenWorktreePaths at internal/websterengine/runwrites_test.go:150, the suspect_test.go fixture for trackedRel and suspectBlobs, the TestFailBatch_RecordsSuspectBlobs fixture in batchfail_integration_test.go for failBatch). In each row make PaneCwd a subdirectory of WorktreeRoot, pass a path relative to PaneCwd, and assert that it resolves against PaneCwd while the within-tree check, the returned repository-relative path and the git probe use WorktreeRoot (a recorded blob or path of the form backend/<file>). Add rows to the existing tests and add no new test functions.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/webstercli/verbs_test.go:89
    summary: The begin-batch and record-batch tests run in a non-dot anchor hub but their plan has no root, so the dispatch re-resolution in BeginBatch, the done checks and drift in RecordBatch and DiskSizes can resolve against the anchor again without any test failing
    fix: In newVerbsFixture (internal/webstercli/verbs_test.go:89) seed the plan with Root set to the fixture's anchor directory (backend) and card targets that exist under it, instead of the root-less plan from seedValidPlanDir. Then TestBeginBatchCmd_HappyPath fails with path-missing at the dispatch boundary (beginbatch.go:480) and TestRecordBatchCmd_Envelope fails in DoneChecks (recordbatch.go:933) when either call is moved back to the anchor. Adjust the committed Create target in TestRecordBatchCmd_Envelope to the same root-prefixed path.
---

### [MEDIUM] Two-root helpers have no split-root test

**Location:** internal/websterengine/suspect.go:268, suspect.go:319, runwrites.go:72, batchfail.go:93
**Issue:** The diff gives `trackedRel`, `suspectBlobs`, `writtenWorktreePaths` and `failBatchInput` two roots, with a relative path resolved against `PaneCwd` and the within-tree check, the repository-relative path and every git probe on `WorktreeRoot`.
Every engine fixture the diff touches sets `PaneCwd` equal to `WorktreeRoot`, and `runwrites_test.go:150` passes `worktree, worktree`.
A change that swapped the two arguments, or read the wrong one, passes every engine test.
The webstercli verb tests do run with distinct roots, but their audit is faked, so they reach none of these paths.
The behaviour is the diff's own addition and nothing pins it.

### [MEDIUM] Dispatch, done-check and size paths are not pinned by a root plan in a non-dot anchor hub

**Location:** internal/webstercli/verbs_test.go:89
**Issue:** The decision record asks for a test that runs begin-batch and the plan path checks in a non-dot-anchor hub with a `root:` plan.
`TestPersistentPreRunE_PlanDirAnchoredAtSubpath` now carries that plan, which pins `validate`.
The begin-batch tests run in the same kind of hub, so #32 is pinned, but their plan comes from `seedValidPlanDir` and has no `root:`.
`index.ValidateDispatch` at `beginbatch.go:480` and `runlevel.go:648`, `DoneChecks`, `RebindResigns`, `Delta` and `DetectDrift` at `recordbatch.go:933-1067`, and `DiskSizes` at `cli.go:292`, `runlevel.go:511` and `rebaseline.go:110` are the same #31 fault at other call sites.
Moving any of them back to the anchor leaves the suite green.

## Rejected

- Stencil prose calls `{{.worktree_root}}` the worktree root while it is filled from `PaneCwd` (lens:code-affected-set, LOW): the fill is unchanged by the diff, the record scopes no stencil change, and the geometry doc states the token's source.
- One-line `PaneCwd` additions to engine fixtures carry no assertion that depends on them (lens:code-tests, LOW): where the code under test reads `PaneCwd` the field is required for correctness, and where it does not the line is harmless.
- `readModulePath(WorktreeRoot)` now reads `go.mod` at the git root, not the anchor (lens:code-correctness, observation): `commitChangedPaths` returns repository-relative paths, so the git root is the consistent base, and the record names `go.mod` as a worktree-root file.
- `TestWire_ModeHubSelectsHubMode` repeats the `hubgeom` geometry assertion (lens:code-tests, no finding raised): it pins the wiring of the geometry, not a second copy of the constructor's behaviour.
