# burlerengine: let forks run git in repositories they created

The cluster and twill fix rounds fail a fork that runs any state-mutating git command.
A fork that built a throwaway repository in its scratchpad to test git behaviour (`git init`, an empty commit, `git worktree add`, `git worktree repair`) therefore halted the whole run, though no target state was at risk.

The audit in `internal/burlerengine/cluster.go` now walks a fork's Bash commands in transcript order and records each repository the fork creates: the directory of a `git init`, and the path of a `git worktree add` run inside a recorded repository.
A mutating git invocation whose directory lies inside a recorded repository no longer counts.
A fork that mutates any other repository fails the round exactly as before, with the same error message.

Closes #843

## Design decisions

- The transcript carries no working directory, so the invocation directory is resolved lexically.
  It starts from the command's unknown starting directory, moves through each earlier `cd <path>` segment of the same command, then through each `-C <path>` option.
  A `cd` in an earlier command never carries over.
  An unresolvable directory matches no recorded repository, so the check errs toward flagging.
- These cases are never excused, since they can reach the target's repository from inside a recorded directory:
  - `git push`;
  - an invocation with `--git-dir` or `--work-tree`;
  - an invocation after a bare `cd` or `cd -`;
  - a `..` that climbs above a leading `~` or `$` word.
- A `git init` records no repository when its relative path is the starting directory, an ancestor of it, or leaves it through `..` (`git init`, `git init ..`), since that may reinitialize the target.
- `git clone` does not record a repository; the issue names only `git init`, `-C` and `cd`.
- The alternative from the issue, downgrading the violation to a recoverable stuck state, is not taken.
- `commandRunsMutatingGit` and `gitSubcommand` are replaced by `firstMutatingGitCommand` and a `gitInvocation` parser, which keep the existing verdicts for the mutating-git matrix.
  `internal/burlerengine/doc.go` describes the new rule in the fork-discipline paragraph.

## Known gaps

- A fork that reinitializes the target by an absolute path (`git init /abs/target`) and commits there passes the audit, since the transcript does not reveal the target's absolute path.
  The fork stencils still forbid it.
- A `cd` inside a subshell, as in `(cd x && git commit)`, is not tracked, so such a command against a fork-created repository still fails the round.

## Tests

`internal/burlerengine/cluster_test.go` adds table rows for the issue's command shape, cross-command `git init` then `git -C`, `cd <dir> && git commit`, worktrees added from a created repository, a sibling sharing a created directory's name prefix, and each never-excused case above.
They are tier-free and run no git.
No manual check is outstanding.
