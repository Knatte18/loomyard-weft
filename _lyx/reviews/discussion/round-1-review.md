---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (D1 rationale, Open risks)
    summary: merge-in runs git merge --ff, so a fast-forward moves HEAD by non-merge parent commits that D1 refuses, re-creating the wedge; the record claims merge-in adds exactly merge commits
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (Open risks item 2); support-log.md (Rejected alternatives)
    summary: the second-parent open risk names the fork audit as the control, but CheckFork explicitly allows repo-native git, so a fork's own post-report git merge is unpoliced
  - id: F3
    severity: LOW
    location: _lyx/discussion/decision-record.md (D4)
    summary: D4 does not say how MERGE_HEAD is probed or on which side, and its lyx fabric pointer is wrong for a standalone run
  - id: F4
    severity: LOW
    location: _lyx/discussion/decision-record.md (D6)
    summary: D6 says the resolved anchor without naming the Location's AnchorPath, the value hubgeom.WebsterGeometry hands webster as AnchorRoot
  - id: F5
    severity: LOW
    location: _lyx/discussion/decision-record.md (D6)
    summary: D6 does not say which merge-in envelopes carry the warning; the conflict result, the path that leads into D4, is left open
  - id: F6
    severity: LOW
    location: _lyx/discussion/decision-record.md (D2, Open risks)
    summary: D2 gives no rationale for tree checks reading the merged tree, and a parent-side rename falls in no batch delta, so drift never auto-repairs it
  - id: F7
    severity: LOW
    location: _lyx/discussion/decision-record.md (Acceptance criteria)
    summary: no criterion covers D6's predicate-error degradation, and the two-merge criterion leaves D3's warning content implicit
  - id: F8
    severity: NIT
    location: _lyx/discussion/decision-record.md (Auto-mode assumptions); support-log.md (Q12)
    summary: the envelope key is left to Plan although webster's CLI envelopes already carry a warnings array, so the codebase answers it
---

### [MEDIUM] A fast-forward merge-in moves HEAD by non-merge commits

**Location:** `decision-record.md`, D1's rationale and `## Open risks`.

**Issue:** D1's rationale says "a parent merge-in adds exactly merge commits on the task branch's first-parent line".
`fabricengine.MergeIn` calls `gitrepo.MergeStart(sha, false)`, which runs `git merge --ff --no-commit` (`internal/gitrepo/merge.go`), and `--no-commit` does not suppress a fast-forward.
When the report's `head_sha` is an ancestor of the parent's tip, the merge-in fast-forwards: HEAD lands on the parent's tip, and the first-parent walk back crosses the parent's own ordinary commits.
D1 refuses that, and the run is wedged exactly as today.
The shape is narrow but reachable: the task's warp carries no commits the parent lacks (loom's pre-Webster artifacts live on the weft), and the batch's fork committed nothing (a batch whose report is OK with `head_sha` equal to `StartSHA`).
The other shape the focus asked about is fine: a conflicted merge-in concluded through `lyx fabric merge --continue` lands through `MergeConclude`, a `git commit` with `MERGE_HEAD` live, so it is a two-parent merge commit D1 admits.

**Fix:** Correct D1's rationale to say a merge-in adds merge commits whenever the task's HEAD has commits the parent lacks, and fast-forwards otherwise.
Keep D1 as decided (admitting a fast-forward would admit a fork's own post-report commits, the rejected "any descendant" alternative), add the fast-forward shape to `## Open risks` with its recovery, and pin the refusal with an acceptance criterion.

### [MEDIUM] The fork audit does not police a fork's own git merge

**Location:** `decision-record.md`, `## Open risks` item 2; `support-log.md`, the "Verify each merge's second parent" rejected alternative.

**Issue:** Both say the fork audit is the control on a fork running `git merge` after its report.
`internal/websterengine/audit.go` says the opposite: `CheckFork` bans only Agent calls, fabric references, and writes to the two contract files, and "repo-native git are explicitly allowed".
A fork that writes its report and then runs `git merge <branch>` passes the audit and D1 alike, and its merged content lands in the next batch's `StartSHA` without appearing in any batch's delta.
The residual risk is real and uncontrolled, and the stated mitigation is false.
The item also re-litigates the rejected alternative's deliberation, while the requirement the plan writer needs (the second parent is not checked) already sits in `## Auto-mode assumptions`.

**Fix:** Rewrite the open risk to state the residual acceptance and its only visibility (D3's warning names every walked merge SHA), without the false control and without the deliberation.
Correct the support-log's rejected alternative to the real reason: `websterengine` does not know the parent branch.

### [LOW] D4 leaves the merge-state probe and its pointer open

**Location:** `decision-record.md`, D4.

**Issue:** "`MERGE_HEAD` present" does not say how to probe it.
In a linked git worktree `.git` is a file, and `MERGE_HEAD` lives in the per-worktree git dir, so a plan that stats `<worktree>/.git/MERGE_HEAD` never fires.
It also does not say which side: the head cross-check reads `WorktreeRoot`'s repo, the warp, and merge-in never merges the weft (`MergeIn`'s `WeftOutcome` is always already-up-to-date), so the warp is the side that matters.
The error's pointer names `lyx fabric merge --continue`/`--abort`, which does not exist for a standalone run with no fabric.

**Fix:** State that the probe asks git for `MERGE_HEAD` in the repo at `WorktreeRoot` (the warp), through `gitrepo`, never a file stat; and that the pointer names the fabric verbs in hub mode and plain `git merge --continue`/`--abort` otherwise, or both.

### [LOW] D6 does not name the anchor it hands the predicate

**Location:** `decision-record.md`, D6.

**Issue:** "the resolved anchor" could be read as the worktree path, the warp path, or the anchor path.
Webster's `Dir` is built from `l.AnchorPath()` in hub mode (`hubgeom.WebsterGeometry`), and in a subpath-anchored hub the anchor path and the worktree path differ, so the wrong choice makes the warning never fire.
The `merge-in` RunE already has the `*lyxcwd.Location` its `PersistentPreRunE` resolved.

**Fix:** State that D6 passes that Location's `AnchorPath()`, the same value `hubgeom.WebsterGeometry` hands webster as `AnchorRoot`, so the predicate reads the state of the run in the worktree being merged into.

### [LOW] D6 does not say which envelopes carry the warning

**Location:** `decision-record.md`, D6.

**Issue:** `setMergeExit` emits three envelopes: ok, conflict, and failure.
A conflicted merge-in is exactly the path that leaves the pair mid-merge and trips D4, so the operator most needs the warning there, yet D6 names only "the merge-in result envelope".
An already-up-to-date result moves nothing, so a warning there is noise.

**Fix:** State that the warning rides on the ok envelope when the merge moved HEAD and on the conflict envelope, and not on an already-up-to-date result or a hard failure.

### [LOW] D2's tree-check half has no rationale, and a parent-side rename escapes drift repair

**Location:** `decision-record.md`, D2 and `## Open risks`.

**Issue:** The pairing is coherent, not a contradiction: the delta and drift's triggering SHA are the batch's own work, and `DoneChecks` and drift's post-repair resolve validate against the tree the remaining plan will build on.
D2 states the split but justifies only the SHA half, so a plan writer could "fix" the tree half to read `head_sha`.
The split has a consequence the record does not list: a symbol the parent renamed or deleted falls in no batch's delta (this batch's ends at `head_sha`; the next batch's starts at the merge commit), so drift never auto-repairs it.
It surfaces only as a blocking finding from the next `begin-batch`'s `ValidateDispatch`.

**Fix:** Add the tree-half rationale to D2 and the parent-side-drift consequence to `## Open risks`.

### [LOW] Acceptance criteria miss D6's degradation path and D3's two-merge content

**Location:** `decision-record.md`, `## Acceptance criteria`.

**Issue:** D6 decides that a predicate error degrades to no warning plus a `logger.Warn` and never fails the merge; no criterion tests it.
"Same with two successive merge commits" leaves open whether the single warning names both merge SHAs, which D3 requires.

**Fix:** Add a criterion for merge-in with a malformed `outcome.yaml` (merge completes, no envelope warning), and spell out that the two-merge warning names both merge SHAs.

### [NIT] The envelope key is answerable from the codebase

**Location:** `decision-record.md`, `## Auto-mode assumptions` item 4; `support-log.md`, Q12.

**Issue:** Webster's own CLI envelopes (`internal/webstercli/recordbatch.go`, `recoverbatch.go`) already put warnings under a `warnings` string array, so leaving the key to Plan defers a question the code answers.

**Fix:** Decide the key as `warnings`, following webster's CLI envelopes, and close Q12.

## Verdicts on the focus items with no finding

- **Weft side of the parent merge:** not a defect and outside this task.
  `MergeIn` never merges the weft (its record's `WeftOutcome` is fixed at already-up-to-date), so a parent merge-in cannot touch `_lyx/webster/` state or reports.
- **D6's import:** confirmed; `go list -deps ./internal/fabriccli` already includes `websterengine`, and `websterengine` imports `fabricengine`, not `fabriccli`, so no cycle.
- **Relocation beyond F2:** the other open risks and assumptions each carry a requirement or a residual risk the plan writer needs; nothing else moves.
