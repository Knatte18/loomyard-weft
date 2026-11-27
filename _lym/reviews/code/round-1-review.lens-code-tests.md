```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test that pins it.

**Per behaviour, the test that pins it**

1. The remote tip moves to a commit the start head lacks, so the run ends `pushed` with the new wording, the `startHead..remoteTip` range and no "pushed" in the text. Pinned by `TestFixerRun_Outcomes` row "…to a commit the start head lacks is the pushed end" (`internal/landingshed/fixerrun_test.go:339-357`).
2. The remote tip moves to a commit the start head holds, so the run does not end `pushed`. Pinned by the row "…to a commit the start head holds is no push" (`fixerrun_test.go:359-369`). The start head is read before the session: the turn moves `fx.head` to `f1`, so a late read would query `head-1..f1` and the row would fail. This row cannot catch swapped arguments to `IsAncestor`, because sha equals ref (`head-1..head-1`). See F1.
3. The remote ref is deleted, so the run ends `pushed` with the delete wording. The row "a remote ref deleted…" (`fixerrun_test.go:371-379`) pins the cause. The delete way forward ("check why the task branch is gone from origin") is not asserted.
4. An `IsAncestor` failure returns an error. None. See F2.
5. `IsAncestor` is skipped when the tip is gone (`remoteAfter == ""`). Pinned by the delete row: with the fixture's map, a call would answer false and fall through to the same end, so the skip itself is not distinguishable. Acceptable.
6. `sessionRef` gives the engine and session id. Pinned by every halt row through `fakeFixerSession` (`fixerrun_test.go:257,269,280,298,325,351,378`).
7. `sessionRef` names the run dir only while it exists. Pinned by the `keepRunDir` timeout row together with the named / not-named assertions over all rows (`fixerrun_test.go:254-258`, the run-dir checks after the reason checks).
8. `sessionRef` answers "none recorded" when it has no id and no existing dir, and drops the leading space when Engine is empty. None. See F3.
9. Fix review no-verdict text uses "session:". Pinned by `finalize_gate_test.go:851,864`.
10. The fix review "changed the task branch" halt now uses `session:` (`internal/landingshed/finalize.go:1144`). None: row `finalize_gate_test.go:883` does not assert the session part. See F4.
11. The `gitrepo` trace line on a plain branch push. Pinned by `TestPushBranch_LeavesTraceLine` (`internal/gitrepo/pushbranch_integration_test.go:23-41`).
12. The trace line on a delete (`""` tip), a leased update/delete and the rebase-retry push. None. See F5.
13. The real `IsAncestor` wiring in `internal/loomcli/landingdeps.go:346-352`, and its argument order through to fabric. None: it is only listed in `intentionallyNil`. See F6.
14. The seam-list and `envkit` entries. Pinned by the existing wiring tests.

**Findings**

- **F1** — MEDIUM, design (test economy: uncovered behaviour). `internal/landingshed/fixerrun_test.go:359-369`.
  - Summary: no row pins the argument order of the ancestry query, so a swap to `IsAncestor(startHead, remoteAfter)` passes every row. The no-push row uses a tip equal to the start head (`fx.ancestors = {"head-1..head-1": true}`), so the order is invisible.
  - Evidence: `fixerrun.go:193`; fixture `fixerrun_test.go` IsAncestor at line ~170.
  - Fix: add or change a row where the tip is a strict ancestor of the start head, for example `remoteTip = "older"` with `ancestors = {"older..head-1": true}`.
- **F2** — LOW, design (uncovered behaviour). `internal/landingshed/fixerrun.go:193-195`.
  - Summary: the error path when the ancestry query fails has no row. That covers the returned error, the attempts record returned as `session.rec`, and the absence of a `pushed` end.
  - Evidence: the fixture's `IsAncestor` always returns a nil error (`fixerrun_test.go`, `newFixerRunFixture`).
- **F3** — LOW, design (uncovered behaviour). `internal/landingshed/fixerrun.go:232-246`.
  - Summary: the `sessionRef` fallback "none recorded", and the empty-Engine trim, are reached by no test. `fakeFixerSeats.chair` always sets both `SessionID` and `Engine` (`fixerrun_test.go:66-73`).
- **F4** — LOW, design (uncovered behaviour). `internal/landingshed/finalize.go:1144`.
  - Summary: the changed-branch halt's new `session: &lt;ref&gt;` part is not asserted. Row `finalize_gate_test.go:883` checks only the cause, the path and the way forward.
- **F5** — LOW, design (uncovered behaviour). `internal/gitrepo/push.go:183`.
  - Summary: the trace line is pinned only for `PushBranch`. The delete paths (`push.go:344,381`), where the tip is `""`, and `UpdateRemoteBranchLeased` (`push.go:414`) have no assertion on the line. The diff's own doc (`gitrepo/doc.go`) promises it for "every push or remote-branch delete".
  - Evidence: `pushbranch_integration_test.go:23-41` is the only test that reads "push landed".
- **F6** — LOW, design (uncovered behaviour). `internal/loomcli/landingdeps.go:346-352`.
  - Summary: the production `IsAncestor` closure is reached by no test. It covers opening fabric over the task pair and `Fabric.IsAncestor` on the warp checkout, with SHA validation of a remote tip. It is listed only as intentionally nil in `wiring_seams_test.go:36`.

Tool calls:
1. Grep of landingshed tests for "changed the task branch", `fakeFixerSession` and "none recorded", and of all tests for "push landed". My context held the diff hunks but not the unchanged changed-branch test row or the full set of trace-line assertions.
