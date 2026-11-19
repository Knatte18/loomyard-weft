---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/runfsm/act.go:147
    summary: "Owed records pushes prescribe a recovery command in a directory where that command is refused."
    fix: "Restore the general PushOwedWayForward propagation through runfsm Inputs, Judge and Verdict and pairteardown.ReadRun, obtaining it from FailedPushWayForward. Use that resolved instruction in Verdict.Refusal while keeping the retired failure classes removed. Correct existing owed-push assertions to expect the code anchor for records-side and subpath-anchored failures."
    origin: lens:code-affected-set
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/fabricengine/boardweft.go:39
    summary: "A remote-only board branch becomes an empty orphan during a two-repository clone."
    fix: "Restore ensureBoardWorktree's adoption of origin/warpBranch with a tracking local branch when the local branch is absent. Create an orphan only when both local and remote board refs are absent. Correct the existing clone-adoption test to exercise different code and records default branches and assert preservation of the remote board history."
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: scope
    location: internal/loomcli/wiring_commitstatus_integration_test.go:367
    summary: "The removal deletes the retained two-repository cherry-pick transition regression without replacement."
    fix: "Restore the records cherry-pick in progress commits the transition row of TestCommitStatusSeam_Real and its necessary fixture helpers from the review base. Preserve its first-probe cherry-pick abort and assertions that records HEAD advances with the status paths. Keep the following one-repository wait rows deleted."
    origin: lens:code-scope
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/fabricengine/clone.go:275
    summary: "Clone tests do not detect removal of post-checkout hook installation."
    fix: "Extend an existing successful clone test to read the returned code checkout's post-checkout hook and assert the fabric hook content and, on platforms requiring it, executable mode. Do not invoke InstallPostCheckoutHook in that test before the assertion."
    origin: lens:code-tests
  - id: F5
    severity: BLOCKING
    class: consistency
    location: internal/fabriccli/cli_test.go:407
    summary: "Flattening the clone-refusal test drops required parallel execution."
    fix: "Add t.Parallel immediately after tiergate.Require in TestCommand_CloneEngineRefusalReachesEnvelopeUnchanged."
    origin: lens:code-tests
---

Reviewed `7346d6a8c8b38d0b26987969ac76b6b3843727c6..91b285daca36b5631feb9b4752bea72bbee6346f` against the decision record, parent amendment, focus and review rubric. The four lens reports are saved alongside this report.

### [BLOCKING] Owed-push recovery points at the records sibling

**Location:** internal/runfsm/act.go:147

**Issue:** Removing `PushOwedWayForward` makes the refusal render `lym fabric push` in `PushOwedWorktree`. `pairteardown.ReadRun` fills that field from `owed[0].RepoPath` at `internal/pairteardown/runread.go:238`. Records failures store the records sibling there, with `CodeWorktree` stored separately (`internal/fabricengine/pushanchored.go:308`). Ordinary push calls `resolveWarpLocation`, whose `RequireWarpWorktree` rejects that records sibling. The operator therefore follows the instruction into another refusal. Subpath-anchored code failures also require the code anchor rather than the repository root. The surviving `FailedPushWayForward` already resolves these distinctions. This violates PATTERN-refusal-way-forward and the refusal contract's requirement that the prescribed verb make progress. Loom consumes the faulty refusal through `internal/loomcli/ownercheck.go:186`.

### [MEDIUM] Clone discards a remote-only board branch

**Location:** internal/fabricengine/boardweft.go:39

**Issue:** Consider a code remote defaulting to `main` and a records remote defaulting to `records-default`, with existing remote branches `main` and `main-weft`. Cloning creates local `records-default`; suffixing the primary records branch adopts `main-weft` but leaves board `main` remote-only. `BranchExists` checks only `refs/heads`. The changed `ensureBoardWorktree` therefore creates an orphan instead of adopting `origin/main`, returning a board checkout without its stored entries and configuration. `CloneAndWire` then reconciles configuration against that empty board. The deleted remote-ref adoption supported ordinary two-repository inputs and was not specific to the retired layout.

### [MEDIUM] The retained cherry-pick transition test was deleted

**Location:** internal/loomcli/wiring_commitstatus_integration_test.go:367

**Issue:** The deletion hunk beginning at old line 393 removes the two-repository `records cherry-pick in progress commits the transition` row along with one-repository rows. It exercises real status dependencies, observes a stopped cherry-pick in the first probe, ends that pick before the commit takes the write lock, and asserts a new records commit containing the status paths. This pins the retained acceptance criterion and the behavior from parent amendment c83fec49. `TestMergeStateActive` only asserts the probe result and cannot detect a downstream skipped transition. There is no replacement or `Loosened-Check:` trailer. This consolidates the scope and tests lenses' findings about the same deletion.

### [MEDIUM] Hook installation can disappear without failing clone tests

**Location:** internal/fabricengine/clone.go:275

**Issue:** The changed clone wiring resolves the worktree and calls `InstallPostCheckoutHook`. The reported mutations can remove that wiring without failing tests. Existing hook tests call the installer themselves before inspecting the hook, while successful clone tests do not assert its presence. Hook installation is an observable clone contract that an existing clone test can reasonably check; the mutation survivor is therefore a coverage gap.

### [BLOCKING] An isolated clone test no longer runs in parallel

**Location:** internal/fabriccli/cli_test.go:407

**Issue:** `TestCommand_CloneEngineRefusalReachesEnvelopeUnchanged` now proceeds from the tier gate into temporary-directory fixtures and explicit `--into` destinations without `t.Parallel`. The former table and its rows ran in parallel. The retained test does not mutate process-global state and names no exception. This violates PATTERN-test-speed's requirement that tests run in parallel unless they touch documented process-global state.

## Rejected

- code-correctness's proposed stopped-cherry-pick commit defect: a permanently stopped operator cherry-pick can reject a path-limited Git commit, but the retained synchronized replay case ends the pick before the commit acquires the write lock. The requested restoration uses the earlier ordinary commit path; this is not established as a new production regression. F3 preserves the intended transition test without reintroducing the removed temporary-index mechanism.

## Validation

Source and historical-code inspection support these findings. The targeted tier0 gate could not start: creating its holder under the hub's `_bolt/.lym/gate` failed because that directory is read-only in this session. No passing test run is claimed. No target files were changed.
