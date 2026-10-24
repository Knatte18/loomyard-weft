# burler: excuse fork git mutations inside repositories the fork created

The burler fork audit refused any fork that ran a state-mutating git subcommand, whatever repository the command acted on.
A twill fixer fork that built a throwaway repository in its scratchpad (`git init`, an empty `git commit`, `git worktree add`, `git worktree repair`) therefore failed `Plan-Burler-Fix` as a producer hard failure, and the advised recovery had nothing to revert (#843).

## What changes

`forkViolation` now classifies each fork's Bash commands as one sequence through `firstMutatingGitCommand` in `internal/burlerengine/cluster.go`, replacing the per-command `commandRunsMutatingGit`.
The walk records each repository the fork creates: the directory of a `git init`, and the path of a `git worktree add` run inside an already recorded repository.
A mutating invocation whose directory lies inside a recorded repository no longer counts.

The directory is resolved lexically, with no filesystem access:
each command starts from the fork's unknown starting directory, moves through its own `cd` segments, then through the invocation's `-C` options.
A path led by `/`, `~` or `$` is anchored as written, and `..` never climbs above its `~` or `$` word, so `"$S/repo"` matches across commands by spelling.

The excusal is bounded so a mutation can still not reach the target's repository through it:

- `git push` always counts, since it writes to another repository.
- An invocation carrying `--git-dir` or `--work-tree`, or run after a bare `cd` or `cd -`, always counts.
- A `git init` naming the starting directory or an ancestor by a relative path, such as `git init` or `git init ..`, records nothing, since it may reinitialize the target.
- A `cd` in one command does not carry into the next.
- A worktree added from a repository the fork did not create is not recorded, since it shares that repository's refs.

Every other verdict of the existing classification is unchanged, and so are the violation message and its way forward.

## Design decisions

- Scope awareness was taken over the board entry's alternative of downgrading the violation to a recoverable stuck state: it removes the false failure instead of softening it, and needs no before-and-after snapshot of the target.
- Scope is keyed on repositories the fork created, never on "outside the target worktree": the transcript carries no target path, and a worktree of the target placed elsewhere shares its refs.
- `git clone` destinations are not recorded; only `git init` and `git worktree add` create a recorded repository.

## Known limits

- A fork that reinitializes the target through an absolute path, as in `git init /abs/target`, and then commits there passes the audit; the fork stencils still forbid it.
- A `cd` inside a subshell, as in `(cd x && git commit)`, is not tracked, so such a command against a fork repository still fails the round.

## Tests

`TestFirstMutatingGitCommand` keeps the existing single-command matrix, now run through `firstMutatingGitCommand`.
`TestFirstMutatingGitCommandScopesToForkRepositories` covers the issue's command shape, cross-command and `cd` reach, worktrees added from a created repository, and every bound above as a row that must still be returned.
`internal/burlerengine/doc.go` describes the excusal in its fork-discipline paragraph.
