---
round: 1
focus:
  - "internal/fabricengine/configchanges.go `showFile`: does it answer exactly what `git ls-tree` plus `git show rev:path` answered, for a rev that is a ref name, a short SHA, an annotated tag, a path that is a directory or submodule, and a missing path?"
  - "internal/fabricengine/configchanges.go `showFile`: is the full-SHA test (hex of length 40 or 64) sound, and does a failed `ResolveSHA` return an error where git returned one?"
  - "internal/gitkit/query.go `headCommit`: does `RevParse(tb, dir, \"HEAD\")` return what `git rev-parse HEAD` printed in a linked worktree, a worktree reached through a symlink, a subdirectory of a worktree, a detached HEAD, a bare repository and an unborn branch?"
  - "internal/gitkit/query.go `headCommit`: can `dotgit.ListWorktrees` report a stale HEAD, or a Head that is not a full commit SHA, so the Go read differs from git?"
  - "Scope against the decision record: the record limits changes to `internal/fabricengine` and `internal/gitkit`; does the diff touch only those and `pattern/`, and does every change replace a read-only git call without changing what a production path decides?"
  - "Declared PATTERN change: the `gitkit` import list in `pattern/PATTERN-leaf-packages.md` now includes `dotgit`; check `internal/gitkit/leaf_enforcement_test.go` and the pattern text agree and that `dotgit` itself imports nothing that makes `gitkit` reach a feature package."
  - "The record asks for a count, a replacement and a before/after measurement of tier 1 wall and git process count: check `_lym/landing/summary.md` records both measurements, and that its claims match the diff."
  - "Test economy: do `TestShowFile_ReadsCommittedBytes` and the new `gitkit` tests cover behavior new to this diff without duplicating an existing test, and is the `RevParse` fallback to git (bare, unborn, non-HEAD rev) covered?"
---

Seed focus for round 1, drawn from the diff `bf05b2ef..HEAD`: two read-only git calls moved to Go (`showFile`, `RevParse` of HEAD).
The main risk is a Go answer that differs from git's in an edge case.
