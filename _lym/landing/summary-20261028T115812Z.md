# Burler fork audit reads a listing git branch as read-only

A burler review fork that ran `git branch -a --contains <commit>` failed its whole review round, because the fork audit in `internal/burlerengine/cluster.go` counted every `git branch` as git-mutating.
`git branch` is now judged by its arguments, as `git tag` already was.

## What changes

`gitBranchReadsOnly` decides whether a `git branch` invocation only lists branches or shows the current one.
It passes when every option is a known read-only option and the invocation either has no operand or carries a list-mode option, which turns operands into patterns.

- Read-only options sit in an allow-list: `-a`, `-r`, `-l`, `-v`, `-i` and their long forms, `--contains`, `--no-contains`, `--merged`, `--no-merged`, `--points-at`, `--show-current`, `--format`, `--sort`, `--column`, `--color`, `--abbrev`, `--omit-empty` and their `--no-` forms where git has one.
- List mode comes from `-l`/`--list`, `-v`/`--verbose`, `--contains`, `--no-contains`, `--merged`, `--no-merged`, `--points-at` and `--show-current`.
- Short-option clusters such as `-avv` are read letter by letter.
- `--format`, `--sort` and `--points-at` given as a separate word skip their value, so `--sort -committerdate` neither reads as an option nor as an operand.

The `firstMutatingGitCommand` doc comment and the fork-audit paragraph in the package doc name the new exception.

## Design decisions

- **Allow-list, not deny-list.** Any option the list does not know, writing or new, keeps the command mutating, so a miss fails the round loudly rather than letting a fork change a branch.
- **`-a`, `-r`, `--format` and `--sort` alone do not make `git branch` read-only beside an operand.** The board entry lists them among the list-mode options, but git only enters list mode for them when no operand is given: `git branch -a foo` creates `foo`. They pass with no operand, and beside an operand only with a list-mode option.

## Tests

`TestFirstMutatingGitCommand` gains rows for the listing forms, including the `git branch -a --contains 71a46b11` that failed the round, and for the writing forms: create, `-d`/`-D`, `-m`, `-c`, `-u`, `--set-upstream-to=`, `--unset-upstream`, `-f`, a name beside `-a` or `--format`, `-d` beside `-l`, `-aD` and a name after `--`.
