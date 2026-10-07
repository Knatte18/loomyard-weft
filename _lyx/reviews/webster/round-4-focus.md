---
round: 4
focus:
  - "In `internal/loomcli/resume.go`, `resumeFromRunState` reads the status file and takes the status lock, and `runLockHeld` probes the run lock; both locks live in the ephemeral scratch directory from `shedrun.ScratchDir`. Settle whether each path creates that directory before the lock is opened, as `approve.go` and `arm.go` do, and whether a worktree with a status file and no scratch directory gets the run's state instead of a lock-open error."
  - "In `internal/loomcli`'s `TestResumeVerb` table, settle whether a row covers a fixture with a status file and no scratch directory, or whether the fix would add a test where a table row would do, against `PATTERN-test-economy`."
  - "In the Go comments and `.md` lines this branch added or changed, settle whether any line still joins independent clauses (a semicolon, or a comma before a coordinating conjunction where each side has its own subject and verb) on one line, against `PATTERN-comment-line-breaks` and the semantic-line-break rule in `CLAUDE.md`. Check `docs/overview.md`, `pattern/PATTERN-completion-signal.md`, `contracts/specs/loom-status-spec.md`, `contracts/stencils/orch/orch-template-role.md` and `internal/loomcli/reviewstatus_test.go`."
---

Focus targets the one open gating design finding and the open consistency residue.
