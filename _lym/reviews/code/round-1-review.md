---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-affected-set
    location: internal/fabricengine/pull.go:461
    summary: "weftDirtyStop and upstreamAnchorStop still name lym fabric commit, pull and anchor in the bare worktree root, which the cwd gate refuses in an anchored hub; the commit message claims every lym step was converted"
    fix: "In internal/fabricengine/pull.go weftDirtyStop wrap f.warpPath in lymVerbDir(f.warpPath) for the `lym fabric commit` step, and in internal/fabricengine/anchorstop.go upstreamAnchorStop (line 170-171) pass lymVerbDir(codeWorktree) as the directory of the `lym fabric anchor --set` step. Both callers (pull.go:199 and pushclassified.go:647) hand it the worktree root. Keep the git steps at the root."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/originreconcile.go:383
    summary: The new lymVerbDir calls in stopWayForward BranchRewritten, anchorStopWayForward, dirtyWayForward (records side) and pull.go settleUnmovedWeft have no test with an anchored worktree, so reverting them breaks nothing
    fix: "Add anchored rows, built on the anchoredHub fixture pattern of TestFailedPushRecord, to the existing tests that assert these way forwards. Cover the code-side BranchRewritten pull line, both anchorStopWayForward variants, the records-side dirtyWayForward commit line, the pull.go records conflict push step and the weftDirtyStop and upstreamAnchorStop lines F1 converts. Extend existing tables or assertions rather than adding new test functions."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/originreconcile.go:383
    summary: The BranchRewritten split on b.Repo has an unreachable records branch, since judgeBranch yields BranchRewritten for code branches only; the leftover line names the records sibling, which refuses fabric verbs
    fix: "In stopWayForward replace the if and the trailing return in the BranchRewritten case with a single return of the lym fabric pull line in lymVerbDir(b.CodeWorktree). For a code branch CodeWorktree equals Worktree."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/failedpush.go:242
    summary: lymVerbDir has no row for an empty worktreeRoot or for an unreadable or invalid marker falling back to the root
    fix: "Add to the way-forward rows of TestFailedPushRecord a records-side row with an empty CodeWorktree expecting the empty directory, and a code-side row whose worktree has an invalid .lym-anchor marker expecting the root."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/fabricengine/doc.go:1462
    summary: The doc addition says every way forward names a lym verb in the anchored directory, but add-time refusals in addparent.go and addadoptonly.go still name lym fabric checkout in a root path
    fix: "Narrow the doc.go sentence to the failed-push notice and reconcile stop way forwards the diff converts, rather than every way forward. Leave addparent.go and addadoptonly.go unchanged, since they are outside the issue."
---

### [MEDIUM] Unconverted lym steps in weftDirtyStop and upstreamAnchorStop

**Location:** internal/fabricengine/pull.go:461, internal/fabricengine/anchorstop.go:170
**Issue:** The issue is a way forward that names the worktree root where the cwd gate wants the anchored directory. The diff converts the push steps, the reconcile stops and `pull.go:442`, and its commit message says every lym step now names the anchored directory. `weftDirtyStop` (`lym fabric commit` in `f.warpPath`) and `upstreamAnchorStop` (`lym fabric anchor --set` in `codeWorktree`) still name the root, and both callers pass the root. They fail as written in an anchored hub, like the notice in the issue.

### [MEDIUM] New lymVerbDir calls have no anchored test

**Location:** internal/fabricengine/originreconcile.go:383
**Issue:** Only `pushStep`, `codeDivergedWayForward` and `rewoundWayForward` have anchored rows. The integration tests that assert the other changed lines run in unanchored fixtures, so they pass with or without the conversion.

### [LOW] Dead records branch of BranchRewritten

**Location:** internal/fabricengine/originreconcile.go:383
**Issue:** `BranchRewritten` is set only for code branches (`judgeBranch`, `originreconcile.go:604` and `:638-639`). The new `if` is always true, and the `else` line is dead and wrong for a records worktree.

### [LOW] lymVerbDir fallbacks untested

**Location:** internal/fabricengine/failedpush.go:242
**Issue:** The empty-root return and the Warn-and-root fallback for an unusable marker are documented in its comment and pinned by no row.

### [LOW] doc.go claims more than the diff does

**Location:** internal/fabricengine/doc.go:1462
**Issue:** "Every way forward" is broader than the failed-push and reconcile lines this change covers.

## Rejected

- Affected-set finding 1, MEDIUM, records-side `BranchRewritten` names `b.Worktree`: the branch is unreachable, so it is no live defect; kept as the LOW dead-branch finding F3.
- Affected-set finding 4, `PushFailureGone` names `lym fabric remove` "run from the prime": the lens did not establish a cwd requirement for the pair path, and the text is outside the issue's push-notice scope.
- Focus question on `lymVerbDir` for a records worktree path, a prime and an absent or unreadable marker: judged correct. It resolves the marker as the cwd gate's `resolveCore` does, and a prime is a hub sibling like any task worktree. A missing marker gives the root and an unreadable one logs and gives the root.
