# Fixer report — discussion round 2

Every finding was checked against the code before its fix: `agentname.StrandNameEnv` (`LYX_STRAND_NAME`) exists and reed exports it in `internal/reedengine/panebin.go`; `internal/cliwire/module.go` has no `-C` resolution and refuses `--target-dir` in hub mode, and mode comes from `preflight.ResolveMode(cwd)`; `internal/verifyrun/procgroup_unix.go` shows the process-group kill pattern; `internal/shuttleengine/claudeengine/background.go` tracks background shells, so a background gate step keeps a turn waiting.
`lyx loom validate-discussion` exits 0 after the fixes.

## Fixed

- **F1 (BLOCKING)**: D10's refusal now also keys on `LYX_STRAND_NAME` parsing through `internal/agentname` as a slug-bearing strand name, wherever the worktree resolves.
  D7's deny now refuses any command naming `LYX_STRAND_NAME`.
  D10's Bound names what still passes: a direct write to the hub's `landing.yaml` outside `lyx config`, and an unset hidden behind a script.
  The Goal now promises only what D10 delivers and points at that Bound.
  Scope, the acceptance criteria (deny row, refusal from the prime's directory under a slug-bearing strand name) and the support log's interview item 7, Q7 and the rejected `lyx config` deny entry were updated to match.
- **F2 (MEDIUM)**: D7 now states that hub membership resolves from the `-C` dir through a new `internal/cliwire` step applying `preflight.ResolveMode`, that this departs on purpose from the cwd-based modules and their hub-mode `--target-dir` refusal, and that the same dir goes to `go test -C`.
  The `-C` acceptance criterion now also checks that `go test` runs in that dir.
- **F3 (MEDIUM)**: D8 now runs every gate step, the orch's pre-push step included, as a background Bash call (`run_in_background`) whose exit the agent waits for, so neither the wait nor the run is under the Bash timeout.
  D5's `cli_wait_sec` rationale now says the wait turns a stalled holder into a named error, and no longer refers to the Bash timeout.
  D7 now states that `go test` runs in its own process group, which `lyx gate test` kills when it is terminated by a catchable signal; its Bound names the uncatchable `SIGKILL` case.
  The support log's interview item 4, Q4 and the rejected unbounded-wait reason were updated, and sizing the wait against a run bound was added as a rejected alternative.
  The acceptance criteria now cover background steps and the group kill.
- **F4 (LOW)**: D11's Bound now says that editing the record can skip, narrow or redirect, and names the forged-identity and rewritten-merge effects, with Publish's full rerun still the guard.
  The reviewer's optional log cross-check was not adopted, because the log path sits in the same editable record; this is recorded as a rejected alternative.

## Disputed

None.

## Deferred

None.
