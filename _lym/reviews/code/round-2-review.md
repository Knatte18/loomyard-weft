---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/loomcli/driftwiring_test.go:32
    summary: TestDriftRequests_JoinOnTheWorktreeRootAndStartInTheAnchor is a per-helper test that TestDriftDeps_RunWhereTheStepsDirResolves already covers through the public seams
    fix: Delete TestDriftRequests_JoinOnTheWorktreeRootAndStartInTheAnchor from internal/loomcli/driftwiring_test.go, drop the now unused time import there and the DriftCaptureRequest and DriftCompareRequest re-exports from internal/loomcli/export_test.go. Keep the integration test, whose "ARelativeDirJoinsTheWorktreeRoot" row fails when driftCaptureRequest or driftCompareRequest swaps or drops WorktreeDir or AnchorDir.
---

### [LOW] Per-helper request test duplicates the integration test

**Location:** internal/loomcli/driftwiring_test.go:32
**Issue:** `driftCaptureRequest` and `driftCompareRequest` are unexported helpers reached only through `driftDeps`.
`TestDriftDeps_RunWhereTheStepsDirResolves` in `internal/loomcli/driftwiring_integration_test.go` drives both seams over a hub anchored at `backend`.
Its `NoDirRunsInTheAnchor` row fails if `AnchorDir` is dropped or swapped.
Its `ARelativeDirJoinsTheWorktreeRoot` row fails if `WorktreeDir` is dropped or swapped.
The new `TestDriftRequests_JoinOnTheWorktreeRootAndStartInTheAnchor` therefore asserts nothing the public-surface test does not, and it needs two exports in `export_test.go` to do it.
The `driftcli` counterpart, `TestWorktreeRequests_JoinOnTheWorktreeRootAndStartInTheAnchor`, is not raised: no other test reaches `driftcli`'s request construction.

## Verdicts on the focus items

- **Baseline prompt (`captureReading`, `BaselineTable`):** not a defect.
  The prompt names the directory from the worktree root, joined by `gateslot.ResolveCommandDir`, and tells a directory outside the root whole.
  An unset `dir` renders the anchor relative to the root, `.` when the anchor is the root, so there are no empty backticks and no mention of the removed base.
  `TestSeatTables` "the capture's run directory is told relative to the worktree root" asserts the text for an empty, a `.`, a relative and an absolute-outside `dir`.
  The contract stencil only interpolates `{{.capture_command}}`, so no stencil prose carries the old base.
- **Side capture refusal:** not a defect.
  `missingDirRefusal` switches on the `siteSideCapture` site and names `lym drift capture --side`, the verb `runSideCapture` serves, with no `lym loom resume`.
  The `RunCapture` and `RunCompare` rows assert the capture's and the compare's own text.
  The side-capture row in `TestRunCapture` asserts the side text and the absence of `lym loom resume`.
- **Request bases (`RunCapture`, `RunCompare`, `RunSideCapture`, driftcli and loomcli constructions):** not a defect.
  Every construction passes `WorktreeDir: WorktreePath()` and `AnchorDir: AnchorPath()`.
  The loomcli integration rows, the engine's capture rows under an anchor at `dotnet` and the side-capture row would each fail on a swap or a drop.
- **Battencli `PairDir` seam and notice:** not a defect.
  `stepPairDirNamesTheTaskWorktree` reanchors the pair at `backend`, asserts the anchor differs from the root and asserts `WorktreePath()`.
  `TestReadNoticeChild` asserts `WorktreePath()`.
- **`copyStagedToHub` and `checkHubModule`:** not a defect.
  The editor-save rows refuse a missing capture dir with the staging copy kept, and allow a save over a file whose dir already failed with the same error.
  The `--set` rows cover the refused drift and gate cases and the accepted existing dir.
  The write check, not `LoadConfig`, refuses a missing dir, as the record's `Added after Discussion` entry decides.

Other checks that found nothing:

- The diff touches no `PATTERN.md` entry and the constraints section declares no changed entry.
- `driftengine` and `configcli` take absolute paths and import no `lymcwd`, so `PATTERN-cwd-resolution` holds.
- The summary's claims match the diff, including the carry-over text and `WithDefaultDir` left alone.
- No commit carries a `Pattern-Departure:` trailer.
