---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/landingshed/fixerrun_test.go#TestFixerRun_Outcomes
    summary: No row pins the argument order of the ancestry query, so a swap to IsAncestor(startHead, remoteAfter) passes every row and would hide a push of the session's own commits.
    fix: "In TestFixerRun_Outcomes, add a row where the remote tip moves to a strict ancestor of the start head, distinct from it: set fx.ancestors = map[string]bool{\"older..head-1\": true} and have the turn set fx.remoteTip = \"older\" after writing fixedReport; want fixerEndFixed, wantSpent 1, wantTurns 1. The swapped query would ask \"head-1..older\", answer false and end pushed, so the row fails."
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/landingshed/fixerrun_test.go#TestFixerRun_Outcomes
    summary: The error path of a failing ancestry query in fixerRun.run (fixerrun.go:193-195) has no test.
    fix: "Add a field to fixerRunFixture, such as ancestorErr error, that the fixture's IsAncestor closure returns; in TestFixerRun_Outcomes or a sibling test, move the remote tip during the turn, set ancestorErr, and assert run returns an error whose text holds \"predates the fixer session\" and wraps ancestorErr (errors.Is)."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/landingshed/fixerrun_test.go#fakeFixerSeats
    summary: "sessionRef's fallback \"none recorded\" for a result with no session id and no existing run dir, and its empty-Engine form, are reached by no test."
    fix: "Let fakeFixerSeats report an empty SessionID and Engine through a field read by chair(), and add a TestFixerRun_Outcomes row (for example the session timeout setup) that sets it and wants \"session: none recorded\" in the reason; optionally a row with SessionID set and Engine empty that wants \"session: session sess-fixer\"."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/landingshed/finalize_gate_test.go#TestFinalize_GateAndFixer
    summary: The fix review changed-the-task-branch halt (finalize.go:1144) now names the session, but its test row asserts only the cause, path and way forward, not the session part.
    fix: "In the TestFinalize_GateAndFixer row whose wantReason holds \"changed the task branch\", add \"session: \" + fakeFixerSession to wantReason."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gitrepo/pushbranch_integration_test.go#TestPushBranch_LeavesTraceLine
    summary: The push trace line is asserted only for PushBranch; the delete paths, which log an empty tip, have no assertion although gitrepo/doc.go promises the line for every remote-branch delete.
    fix: "Turn TestPushBranch_LeavesTraceLine into a table over PushBranch and DeleteRemoteBranch (after pushing the branch first): for the delete row, assert a \"gitrepo: push landed\" line holding \"ref=refs/heads/feat\", \"remote=origin\" and an empty tip field; keep the test non-parallel for the global logger."
---

The review range is `d51900fb264f4423409439bc74805c8a92b0ab77..HEAD`, three commits.
The answer key is the decision record alone, with no `Added after Discussion` entries, so the board entry's goal points for issues #271 and #272 are the measuring stick.

Every goal point is built:
the fixer loop reads the task head before the session and ends `pushed` only for a deleted ref or a tip the start head does not hold;
the halt says the remote ref moved or was deleted and names `<start head>..<tip>`;
`recordLandedPush` logs every landed push and delete of a branch;
and fixer and fix review halts name the engine session id, and the run directory only while it exists.
The `Root-Cause:` trailer on 8562edcea names the defect the diff removes.
No PATTERN change is declared, and the diff breaks no PATTERN entry; the ancestry query goes through fabric.
The changed halt-reason assertions are no looser than before: each names the session id, and every row now also asserts the run directory is named only when it exists.

### [MEDIUM] The ancestry query's argument order is not pinned

**Location:** internal/landingshed/fixerrun_test.go#TestFixerRun_Outcomes

**Issue:** The only no-push row asks `head-1..head-1`, where sha equals ref, so swapping the arguments at `fixerrun.go:193` passes every row.
The swap would answer true for a tip that descends from the start head, such as the session's own pushed commit, and silently drop the halt the change keeps for that case.

### [LOW] The ancestry-error path is untested

**Location:** internal/landingshed/fixerrun_test.go#TestFixerRun_Outcomes

**Issue:** The fixture's `IsAncestor` never fails, so the error return at `fixerrun.go:193-195` and its message are reached by no test.

### [LOW] sessionRef's fallback forms are untested

**Location:** internal/landingshed/fixerrun_test.go#fakeFixerSeats

**Issue:** `fakeFixerSeats.chair` always sets `SessionID` and `Engine`, so the "none recorded" answer and the empty-Engine trim in `sessionRef` (`fixerrun.go:232-246`) run under no test.

### [LOW] The changed-branch fix review halt's session part is not asserted

**Location:** internal/landingshed/finalize_gate_test.go#TestFinalize_GateAndFixer

**Issue:** `finalize.go:1144` changed from `session log: <run dir>` to `session: <ref>`, and its row checks neither.

### [LOW] The push trace line on a delete is unasserted

**Location:** internal/gitrepo/pushbranch_integration_test.go#TestPushBranch_LeavesTraceLine

**Issue:** Only `PushBranch` is checked; `DeleteRemoteBranch` and `DeleteRemoteBranchLeased` reach the same line with an empty tip (`push.go:344`, `push.go:381`), a form no test reads.

## Rejected

- code-tests F6, the real `IsAncestor` closure in `internal/loomcli/landingdeps.go:346-352` untested: it forwards its arguments unchanged to `Fabric.IsAncestor` and `gitrepo.Repo.IsAncestor`, whose own tests cover the git query; the seam list marks it as built in run.go like its sibling closures, and the argument order that matters is pinned by F1 at the fixer loop.
- An "already up to date" push still logging `gitrepo: push landed`: the logged tip equals the ref's tip, so the line names no false move.
