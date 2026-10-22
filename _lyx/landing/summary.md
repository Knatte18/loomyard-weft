# burlerengine: let read-only git tag through the fork git guard

The fork git-mutation audit in `internal/burlerengine/cluster.go` refused every `git tag` by subcommand name.
A fork that ran `git tag --points-at HEAD` to list tags halted the run as a hard failure, and the halt message sent the operator to revert a working tree the fork never touched.

`commandRunsMutatingGit` now judges `git tag` by its arguments, through the new `gitTagListsOnly`.
A `git tag` passes when it is bare, or when its arguments before any `--` hold a list-mode option and no tag-writing option.
`git tag v1.0`, `-d`, `-a`, `-s`, `-f`, and `-l -d v1` stay refused.

Closes #819

## Design decisions

- List-mode options are `-l`, `--list`, `--points-at`, `--contains`, `--no-contains`, `--merged`, `--no-merged` and `-n`.
  Long ones are also recognised in `--name=value` form, and `-n` with an attached count such as `-n5`.
  `--sort` and `--format` are skipped with their value, so a value like `-version:refname` is not read as an option.
- Tag-writing options are `-d`, `-a`, `-s`, `-u`, `-f`, `-m`, `-F`, `-e` and their long forms.
  One of them beside a list-mode option keeps the command refused, so an ambiguous command stays refused.
- A `--list` after `--`, or as the value of `-m`, does not make a creating `git tag` read-only.
- A combined short-option cluster such as `-ln` is not parsed and stays mutating.
  This fails closed.
- This weakens the guard for `git tag` only.
  Every other subcommand keeps the subcommand-only rule the guard was given on purpose, so `branch` and `stash` still count as mutating whatever their arguments, and `remote` and `config` stay outside the set.
  `git branch --list` and `git stash list` in a fork therefore still halt the run; the operator scoped them out of this change.
- The halt message is unchanged.
  With the false positive gone, a refused fork command does write a commit, ref or working-tree change, so "revert what the fork wrote" still names the recovery.

## Tests and docs

`TestCommandRunsMutatingGit` in `cluster_test.go` gains rows for the reported command line and for the read-only and mutating `git tag` forms; its existing rows keep their results.
The fork-audit paragraph in `internal/burlerengine/doc.go` states the `git tag` exception.
