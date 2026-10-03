# Card 5 — mergeresolve: commit the session's tracked edits, halt on untracked files, clear stale reports

**Edit:**
- `internal/mergeresolve/mergeresolve.go`
- `internal/mergeresolve/deps.go`
- `internal/mergeresolve/spec.go`
- `internal/mergeresolve/doc.go`
- `internal/mergeresolve/mergeresolve_test.go`
- `contracts/stencils/landing/landing-template-conflict.md`

**Uses:**
- `internal/fabricengine/mergestage.go`

**Intent:**
D10: `MergeSurface` gains `MergeStageTracked` and `MergeUntrackedFiles` (card 4).
After a conflict session whose markers are gone, `resolveConflicts` first reads `MergeUntrackedFiles`;
a non-empty list aborts the merge and reports stuck naming every file, before anything is staged.
Otherwise it stages the conflicted paths (`MergeStageResolved`), then every tracked modification and deletion (`MergeStageTracked`), then concludes with `MergeContinue`.
A semantic-conflict fix the session made outside the conflicted paths now lands in the merge commit instead of being left as working-tree dirt under verify.

This lets the conflict session change any tracked file inside the merge commit.
Bound: Publish and Finalize verify the merge result through the shared function (card 6), which refuses a dirty tree and runs on any tree that differs from the last verified one, PR-Gate reviews the merged result, and new files stay refused.

D11: `Resolve` removes every `conflict-resolution-r*.md` in its scratch dir at entry, after the in-progress-merge abort and before attempt 1, so a report left by an earlier `Resolve` call no longer trips the spec validator's existing-output refusal.
The report is one call's ephemeral diagnostics.
`spec.go` exposes the report-name prefix the glob uses, so the pattern and the path builder share one spelling.

`landing-template-conflict.md`: edits beyond the listed conflicted paths are allowed only to keep the merged tree building and passing, the session creates no new file, and its report names every file it changed beyond the listed paths.

`doc.go` states the staging rule, the untracked halt and the per-call report cleanup.

Tests in `mergeresolve_test.go` (fake fabric):
a session that edits a tracked non-conflicted file reaches `MergeStageTracked` before `MergeContinue` and resolves;
a session that leaves an untracked file aborts the merge and reports stuck naming it, with neither staging verb called;
a second `Resolve` in the same scratch dir after an earlier conflicting one, whose report is still on disk, succeeds.

**ImpactSummary:** both landing producers' merge-in now commits the conflict session's tracked edits and halts on its new files; the conflict stencil's edit scope widens with a stated limit.
