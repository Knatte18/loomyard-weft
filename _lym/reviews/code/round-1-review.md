---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/runproc/end_test.go:524
    summary: The HubOwner subtest name of TestEndSubject_EndsTheRunsThenTheSubjectAndItsDobbyLast still names TestMap after the merge removed the RoleTestMap fixture record and the role itself
    fix: In internal/runproc/end_test.go, rename the subtest name string TheHubOwnerSubjectLeavesTheHubWidePushesTestMapSlackRelayAndTeardownAlone to TheHubOwnerSubjectLeavesTheHubWidePushesSlackRelayAndTeardownAlone; change nothing else in the test.
---

The range is the merge 4153c5ef, read through `git show --remerge-diff`, and the later commit c1a136c7.
Each focus question has a verdict:

- `internal/orchcli/cli.go` and `start.go`: the settlement keeps both sides.
  `launchOrch` calls the beam's `resumeOrch`, which still ends this worktree's own held orch and resumes again, and it now takes `main`'s `remoteControlName` into `orchSpec`, so the first resume and the retry both carry remote control.
  `endTestMap` and its wiring are dropped, which matches `main` deleting its one consumer in `beamDown` and `runproc.RoleTestMap`.
  Nothing either side added is lost.
- `internal/runproc/doc.go`, `end.go` and `end_test.go`: the doc's hub roles list and its `EndSubject` paragraph match `subjectPassSpared` in `end.go`.
  With the `RoleTestMap` record removed, the test still pins the order (runs, then the subject, then its dobby last) and the spared hub-wide roles; only the stale subtest name in F1 is left.
- c1a136c7 changes only the two lines of the `resumeOrch` doc comment that the merge wrote.
- The remerge-diff touches nothing outside the three conflicted files except `end.go` and `end_test.go`.
  Both edits are needed because `main` deleted `RoleTestMap`, which the beam's `EndSubject` used, and the change description states that.

`go build` and `go vet` of `./internal/orchcli` and `./internal/runproc` pass on HEAD.

### [LOW] Stale TestMap in a subtest name

**Location:** `internal/runproc/end_test.go:524`
**Issue:** The merge removed the `RoleTestMap` record (pid 81) from the `HubOwner` fixture at `end_test.go:479-485`, and `main` deleted the role itself.
The subtest name still claims that the hub owner's pass leaves a test-map process alone.
No row tests that, and no such role exists.
The code-tests lens raised the same defect, and it is merged into this finding.

## Rejected

- No lens finding was rejected.
