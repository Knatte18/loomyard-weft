# Burler fork audit reads a listing git branch as read-only

A review fork that lists branches, as in `git branch -a --contains 71a46b11`, no longer fails the Burler round as a git-mutating fork.
Before, `branch` sat in `mutatingGitSubcommands` with no exception, so any `git branch` invocation failed the whole round, and `lym loom resume` only re-ran it for the next listing to fail it again.

Closes #892

## What changes

In `internal/burlerengine/cluster.go`, `git branch` is judged by its arguments, as `git tag` already is, through the new `gitBranchReadsOnly`.
It passes when every option only reads and it either has no operand or carries a list-mode option.
Every other `git branch` still counts as mutating.
`cluster_test.go` adds rows to `TestFirstMutatingGitCommand` for the reading and the writing forms, and the fork-audit paragraph of `doc.go` names the `git branch` exception beside the `git tag` one.

## Design decisions

- **Allow-list, not deny-list.**
  Only the options named in `gitBranchReadOptions` and `gitBranchReadShortOptions` pass, so an option the audit has not classified keeps the command mutating.
  A read-only option git adds later is flagged until it gets one map entry; the failure is loud.
- **Follows git's own mode selection.**
  Git creates a branch whenever an operand is present and no list-mode option is.
  Only `-l`/`--list`, `--contains`, `--no-contains`, `--merged`, `--no-merged`, `--points-at` and `--show-current` mark list mode.
  `-a`, `-r`, `-v`, `--format` and `--sort` read only without an operand, so `git branch -a foo` and `git branch -v foo` still count, since they create `foo`.
  This is narrower than the board entry, which listed `-v`, `-a`, `-r`, `--format` and `--sort` as read-only unconditionally.
- **Short-option clusters are read letter by letter.**
  `-avv` passes, and `-aD` stays mutating because `D` is not a read-only letter.
- **Valued options skip their value.**
  `--format`, `--sort` and `--points-at` given as a separate word consume the next word, so `--sort -committerdate` is neither an option nor an operand.
  The `--name=value` form is read too.
- **Operands after `--`.**
  A word after `--` is an operand, so `git branch -- new` is mutating and a bare `git branch --` or `git branch -l -- feat*` is not.
- The fork-created-repository exception and the help-request exception apply to `git branch` unchanged.

## To check by hand

Nothing here was run by hand; the new table rows are the coverage.

- `internal/burlerengine/cluster.go:52-65`: `-v` and `--verbose` must not count as list mode.
  `git branch -v foo`, `git branch --verbose foo` and `git branch -vv foo` create a branch in a scratch repository, so each must classify as mutating, while `git branch -v` and `git branch -v -l feat*` must not.
- `internal/burlerengine/cluster_test.go:309`: the rows `git branch --format -x` and `git branch --points-at -x` pin the separate-word value skip, and `git branch --` against `git branch foo --` pins the trailing double dash.
  Confirm each fails when the matching branch of `gitBranchReadsOnly` is removed.
- Optional-value options such as `--abbrev 8` read their separate-word value as an operand and so classify as mutating; this fails safe and no case needs it.
