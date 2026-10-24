```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set — find what the gate-runs-past-failing-tier change forces outside its touched lines (amendment round, parent's `internal/gatecli/doc.go` change merged).

**Per change, the set found:**

- **Run loop and exit-code semantics** (`internal/gatecli/run.go:136-160`: a failed run no longer stops the call, and the exit code is the first failure's).
  - Set: `internal/gatecli/doc.go` (lines 8-10, 71, 251-253), `docs/overview.md:461`, `internal/gatecli/cli.go`, `internal/gatecli/test_integration_test.go:953`.
  - `doc.go`, `overview.md` and `cli.go` all describe the new behavior. `doc.go` agrees with the merged `run.go` on the failure and halt semantics, and `overview.md:461` carries the "no failure when no earlier run failed" qualification for exit 75.
  - Covered.
- **`FailedTier` now returns the first failing tier** (`internal/gateslot/runlog.go:188-190`).
  - Set: the caller `internal/landingshed/verifygate.go:57,96`, the doc `internal/landingshed/doc.go:121`, the test `internal/gateslot/runlog_test.go:119-148`, and `internal/gateslot/doc.go:116`.
  - Wording in the landingshed files and `gateslot/doc.go` already says "first failing tier".
  - Covered.
- **Parent's `doc.go` example change** (`tier1,integration` became `tier1`, and likewise `tier2,tmux` became `tier2`).
  - A grep for `tier1,integration`, `tier2,tmux` and `tier3,llm` finds no other occurrence in `.go`, `.md` or `.yaml` files outside `_lyx` and `.lyx`.
  - The merged example block is consistent and no task text contradicts it.
  - Covered.
- **Search for stale "stops at the first failing" wording about the gate** found none. The remaining hits are unrelated (`destroy_test.go`, `wait.go`, `pairteardown`).
  - Covered.

**Findings:** none.

**Tool calls (1):**
1. `grep` across `.go`, `.md`, `.yaml`, `.tmpl` and `.txt` for stale "first failing tier / stops at the first fail" wording and `FailedTier(` callers. My context lacked a repo-wide search for them.
