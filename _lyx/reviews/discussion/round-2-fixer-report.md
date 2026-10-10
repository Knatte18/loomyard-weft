# Round 2 fixer report — discussion

I checked every finding against the code; each premise held.

## Fixed

- **F1** (BLOCKING, design): confirmed `internal/gitrepo/gogit_test.go` is an in-package `integration` test importing `gitkit`, and `go list -deps ./internal/gitkit` includes `lyxcwd`.
  D12 now puts the `.git` entry reader in a new stdlib-only leaf, `internal/dotgit`, imported by both `gitrepo` and `lyxcwd`, and states why it cannot live in `gitrepo`.
  Scope names `internal/dotgit` (new).
  The support log replaces round 1's false premise with a round-2 ledger entry, and lists the export from `gitrepo`, the external-test-package move and `gitrepo` importing `lyxcwd` as rejected alternatives, each with its reason; the earlier rejection of a separate reader package is removed.
- **F2** (MEDIUM, consistency): confirmed `internal/lyxcwd/leaf_enforcement_test.go` pins the allowlist to `gitexec` and gives the `fabricengine -> logger -> lyxcwd` reason.
  The `PATTERN-cwd-resolution` constraint now says the background line and that allowlist both gain `internal/dotgit` in the same commit, and `dotgit` takes on the acyclicity duty by importing the standard library alone.
  A new `PATTERN-leaf-packages` constraint adds `dotgit` with a test-pinned allowlist.
- **F3** (MEDIUM, design): D12 now defines a usable entry the way git's discovery does.
  A `.git` directory counts only when it is a repository (`HEAD` plus `commondir`, or `objects` and `refs`), and any other `.git` directory is skipped.
  A `.git` file ends the walk and is refused with `ErrNotAGitRepo` when its target is no repository.
  The `lyxcwd.Resolve` criterion gains both cases, each inside an outer repository.
  The support log records the rejected walk-on alternative and the round-2 pick.
- **F4** (MEDIUM, scope): confirmed that `linuxIsWatchdogDaemon` in `smoke_proctree_test.go` calls `linuxPids`, `linuxProcArgv` and `linuxProcCwd`.
  D8 and both Scope lines now move the recognizer together with the `/proc` probes it shares with the leftover scan.
- **F5** (LOW, decision): D3 now says a before-to-after comparison reads the per-subcommand total, with fixture and code summed, which the overlap bias cannot move.
  The census criterion names `rev-parse`'s per-subcommand total.
- **F6** (LOW, consistency): the test-budget constraint now states the rule, that a row rises in the commit whose tests push the package past it, in place of the package list.
  A ledger entry records the choice.

`lyx loom validate-discussion` reports `ok: true` after the edits.

## Disputed

None.

## Deferred

None.
