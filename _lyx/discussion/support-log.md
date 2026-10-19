# Support log — fork git-mutation guard classifies by subcommand

## Interview

A darn run holds no interview; the board entry, merging issues #750 and #751, is the whole statement of the change.

## Rejected alternatives

- Tightening the regex to anchor the verb right after git and its options: skipping value-taking options like `-C <dir>` positionally makes the pattern hard to read and to extend.
- Allowlisting read-only subcommands instead of denylisting mutating ones: it would refuse every unlisted read such as `git rev-parse` or `git ls-files`, widening the guard beyond the task.

## Review rounds

_No rounds yet._

## Question ledger

- Should `git branch` and `git tag` with no arguments, which only list, pass? Self-pick: no change; the task targets verbs in argument position, and the mutating set stays as it is.
- Should `-h`/`--help` after a mutating subcommand pass? Self-pick: yes, per issue #750's naming of `--help` and `-h` as read-only.
