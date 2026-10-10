# Round 3 fixer report

## Fixed

- **F1** (BLOCKING, design): confirmed at `internal/fabricengine/hook.go:64-72`, which asks `rev-parse --git-path hooks` because a common-dir join ignores `core.hooksPath`.
  D12's survival rule now keeps a `rev-parse` whose answer depends on config, naming `--git-path hooks` and why.
  The `fabricengine` acceptance criterion now lets `--git-path hooks` survive beside the disambiguation sites and requires the hook to land in the `core.hooksPath` directory on a repository that sets it.
  The support log gains the rejected alternative (a common-dir join) and a round-3 ledger entry.
- **F2** (BLOCKING, design): confirmed: `internal/loomcli/smoke_helpers_test.go` has `findWatchdogPIDs` and `findDriverPIDs`, and `internal/orchcli/smoke_live_test.go` has `killTreeProcs`, each with its own `/proc` pid listing and argv or cwd read.
  D8 now states a rule: `tmuxkit` exports the watchdog recognizer and the `/proc` pid, argv and cwd probes, and every test-side copy moves onto them, whichever package holds it.
  Scope's `tmuxkit` and test-package lines state the same rule instead of naming `reedcli` alone.
  The support log gains a round-3 ledger entry.
- **F3** (MEDIUM, design): confirmed with git 2.53: a gitfile naming a git dir with `HEAD` and a `commondir` file whose target is missing makes `rev-parse --show-toplevel` exit 128 with `not a git repository`.
  D12's repository test now requires `HEAD` in the git dir and `objects` and `refs` in its common dir, the `commondir` target when that file exists.
  The `lyxcwd.Resolve` acceptance criterion gains the missing-`commondir`-target case, refused with `ErrNotAGitRepo`.
  The support log gains the rejected alternative and a round-3 ledger entry.

## Disputed

None.

## Deferred

None.
