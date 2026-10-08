# Round 3 fixer report — discussion

All seven findings were validated against the code and fixed.
`lyx loom validate-discussion` passes on the result.

## Fixed

- **F1** (BLOCKING): confirmed against `internal/cliwire/doc.go` and the allowlist in `internal/cliwire/seam_enforcement_test.go`, which hold no `preflight`.
  D7 no longer routes `lyx gate` through `internal/cliwire`: `lyx gate`'s own pre-run resolves hub membership, and it states why cliwire does not apply (no `--target-dir`, standalone state, plan or stencils).
  The `PATTERN-told-geometry` and `PATTERN-cli-cobra` Constraints lines were restated to match, and `PATTERN-cliwire-sole-wiring` is named as not applying.
- **F2** (BLOCKING): confirmed against `lyxcwd.Resolve` (exact anchor gate) and `preflight.ResolveMode`'s refuse-on-error contract.
  D7 now resolves the `-C` dir, else the cwd, through `lyxcwd.ResolveWorktree`, which finds the containing worktree from any directory inside it, then `preflight.BoardLyxPresent`.
  A dir in no worktree or no hub runs unslotted.
  An acceptance criterion now exercises `lyx gate test -C <nested module dir>`.
- **F3** (BLOCKING): confirmed that `lyx config` has no `-C` flag (`internal/configcli/configcli.go`) and that `env -i` and similar commands drop the variable without naming it.
  D10's Bound now names the env-clearing route beside the direct file write and the script-hidden unset, and states the refusal guards against a mistaken write.
  The `-C` route wording is gone.
  The Goal now claims only the tree-side guarantee in full and calls the `lyx config` refusal a guard against a mistaken write, pointing at D10's Bound.
  Scope's line matches.
  Keying on process ancestry or a reed marker was recorded as a rejected alternative in the support log, since the direct file write stays open regardless.
- **F4** (BLOCKING): D12 now shape-checks every field before rendering: the merge commit is a full hex name resolving to a merge commit through `internal/gitrepo`'s go-git reads, each identity is a Go test name with no whitespace, quote or shell metacharacter, and the log path lies under Publish's verify log directory.
  A failing field does not render and is logged.
  D11's round gate runs only identities passing the same check.
  D11's Bound now states what still passes after the checks: a well-formed test name that never failed and another real merge commit.
  An acceptance criterion pins the refusals.
- **F5** (MEDIUM): D8 now covers each card `**Verify:**` line and the verify-fix fork's `lyx gate test` runs as background Bash calls with the same slot-busy handling.
  D2's slot-busy exemption and D5's background claim name them, and the acceptance criterion pins both.
- **F6** (MEDIUM): D7 keeps the separate process group and states why: killing the tree without signalling the caller's shell.
  It adds a Linux parent-death `SIGKILL` on the `go` child and a Windows kill-on-close job object.
  The Bound now names what still escapes: under a `SIGKILL` on Linux, test binaries and compilers `go` already started; on another Unix, the whole group.
  The Windows Constraints line and the kill acceptance criterion match.
  Keeping `go test` in `lyx gate test`'s own group was recorded as a rejected alternative.
- **F7** (LOW): confirmed that no stencil under `contracts/stencils/orch/` carries a test or pre-push step.
  The orch stencils were dropped from D8, Scope and the acceptance criterion, and `CLAUDE.md`'s push rule now carries the background `lyx gate test` runs alone.

The support log's Interview entries Q3, Q4, Q7 and Q9 record the round 3 changes, the Question ledger's Q4 and Q7 match, and Rejected alternatives gained the alternatives named above.

## Disputed

None.

## Deferred

None.
