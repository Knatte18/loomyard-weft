```yaml
lens: fix-affected-set
outcome: reported
```

Findings (fix-affected-set):

No findings. Checked F1 and F2, the two findings whose premise is a set. The sets that could hide an uncovered member are covered or out of scope:

- F1: the other `samePath` registration checks are not task-worktree presence checks, so they need no `Prunable` guard.
  - `originreconcile.go:654` is the `wanted` predicate in the registered case. A prunable `own` can't reach it now, and the line 663 loop skips prunable entries anyway.
  - `:676` and `:682` look up records and Bolt worktrees.
  - `addadopt.go:45` is a reuse check on the records worktree.
  - `addadoptonly.go:107` already carries `!entry.Prunable`.
- F2: no hand-formed archive tag remains in non-test code.
  - Remaining `ArchiveTagPrefix` uses at `cleanupremotewarp.go:225`, `remoteleftover.go:219` and `recordsreads.go:51` build prefixes for matching, not tag names.
  - `archive.go:74` is the single declaration of the name format.
  - No `archiveTagTipLength`, `archiveTagSHALen` or `archiveTagName` references remain outside tests, and the gate compiled the tests green.
  - Test files still hand-form `tip[:12]` names (`reportcli/convergence_integration_test.go:142`, `remove_archive_integration_test.go:50`, among others). These were not in the review's scope, and a pinned literal in a test is a legitimate oracle.

Findings left unchecked: F3 and F4 are coverage gaps, not sets under one key, so I did not rerun a search for them.

Tool calls:
1. `grep` for archive-tag construction and stale names in non-test files: the context held only the grep results from the fixer's earlier pass, which did not cover `.md` files or construction patterns.
2. `grep` for `samePath` and registration checks, plus `[:12]` and `ArchiveTagPrefix +` in tests: the context did not show every `samePath` site or the tests' tag forms.
