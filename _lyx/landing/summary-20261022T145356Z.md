# Fork git guard lets read-only git tag forms through

A twill fixer's self-check fork that ran `git -C <target> tag --points-at HEAD` hard-failed a `Plan-Burler-Fix` round as a state-mutating git command.
`commandRunsMutatingGit` in `internal/burlerengine/cluster.go` refused every `git tag`, because `tag` is in the mutating subcommand set and the guard reads only the subcommand word.
The audit now judges `git tag` by its arguments and lets its listing forms through, while a `git tag` that creates, deletes or signs a tag stays refused.

Closes #819

## What changes

- `commandRunsMutatingGit` skips a `git tag` invocation when the new `gitTagListsOnly` reports that its arguments only list tags.
- `gitTagListsOnly` passes bare `git tag`, and a `git tag` whose arguments before any `--` hold a list-mode option and no tag-writing option.
  - List-mode options: `-l`, `--list`, `--points-at`, `--contains`, `--no-contains`, `--merged`, `--no-merged` and `-n`, long ones also as `--name=value`, and `-n` also with its count attached, as in `-n5`.
  - Tag-writing options: `-d`, `--delete`, `-a`, `--annotate`, `-s`, `--sign`, `-u`, `--local-user`, `-f`, `--force`, `-m`, `--message`, `-F`, `--file`, `-e` and `--edit`, long ones also as `--name=value`.
  - The value word after `--sort` or `--format` is skipped, so `git tag -l --sort -version:refname` passes.
- `TestCommandRunsMutatingGit` gains rows for the issue's command line, each listing form, and the mutating forms: create, delete, annotate, sign, force, a `=value` message, a delete beside `-l`, `--list` after `--` or as a `-m` value, and a short-option cluster.
- The fork-audit paragraph in `internal/burlerengine/doc.go` states the `git tag` exception.

## Design decisions

- **Only `git tag` is judged by its arguments.**
  The operator decision on the task keeps the subcommand-only rule for every other subcommand.
  `git branch --list` and `git stash list` still count as mutating.
- **A tag-writing option wins over a list-mode option.**
  `git tag -l -d v1` stays refused, so a mixed or ambiguous command errs on the refusing side.
- **Short-option clusters are not parsed.**
  A cluster such as `-ln` keeps the command mutating, rather than adding a short-option parser for a form forks rarely use.
- **The halt message is unchanged.**
  With the false positive gone, a refused fork command has written a commit, ref or working-tree change, so the twill fixer's way forward, "revert what the fork wrote", still names the recovery.
