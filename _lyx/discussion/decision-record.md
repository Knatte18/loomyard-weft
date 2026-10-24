# Decision record — fork git-mutation guard ignores repositories the fork created

## Goal

A burler fork that runs state-mutating git commands only against a repository it created itself no longer fails the cluster or twill fix round.
A fork that mutates the target's repository still fails it, exactly as today.

## Scope

- `internal/burlerengine/cluster.go`: the per-fork mutating-git classification `forkViolation` applies.
- `internal/burlerengine/cluster_test.go`: coverage of the scoped classification.
- `internal/burlerengine/doc.go`: the fork-discipline paragraph describing what counts as git-mutating.
- Out of scope: the provider transcript audit in `internal/shuttleengine`, the webster audit, the fork stencils, and the alternative of downgrading the violation to a recoverable stuck state.

## Decisions

### A fork's own repositories are tracked across its commands

Decision: the audit walks a fork's Bash commands in transcript order and records each repository the fork creates: the directory of a `git init`, and the path of a `git worktree add` run inside a repository already recorded.
A mutating git invocation whose directory lies inside a recorded repository does not count.
Rationale: the fork's transcript holds no cwd or target path, and a repository the fork itself initialized holds no target state whatever its location, so this scopes the guard without new provider facts.

### The invocation directory is resolved lexically

Decision: the directory a git invocation acts on is the command's starting directory, moved by each earlier `cd <path>` segment of the same command, then by each `-C <path>` global option.
A path starting with `/`, `~` or `$` is anchored as written; any other path joins the current directory.
Each command starts from the fork's unknown starting directory, so a `cd` in an earlier command never carries over.
Rationale: lexical resolution needs no filesystem and errs toward flagging: an unresolvable directory never matches a recorded repository.

### Bounds on what the excusal lets through

Decision: the excusal skips only invocations whose resolved directory lies inside a recorded repository, and never one of these:
- `git push`, which writes to another repository;
- an invocation carrying `--git-dir` or `--work-tree`, whose repository the directory does not name;
- an invocation after a bare `cd`, whose directory is the home directory.
A `git init` is never recorded when it names the starting directory or one of its ancestors by a relative path, such as `git init` or `git init ..`, since that may reinitialize the target.
Rationale: these are the routes by which a mutation inside a recorded directory could still reach the target's repository.

## Constraints

- PATTERN-test-speed: the new coverage is tier-free table rows in the existing internal test package, parallel and hermetic, with no real git.
- PATTERN-gitrepo-client-boundary: the change only classifies command strings and runs no git.
- The error message and the way forward of a real violation stay unchanged.

## Auto-mode assumptions

- A darn run holds no interview; the scope is read from the board entry and the code alone.
- The board entry's primary remedy, scope awareness, is taken; its alternative, a recoverable stuck state, is not.
- `git clone` is not recorded as creating a repository, since the issue names only `git init`, `-C` and `cd`.

## Open risks

- A fork that reinitializes the target by an absolute path, as in `git init /abs/target`, and then commits there passes the audit; the fork stencils still forbid it, and the audit cannot tell the target's absolute path from the transcript.
- A `cd` inside a subshell, as in `(cd x && git commit)`, is not tracked, so such a command against a fork repository still fails the round.

## Acceptance criteria

- The issue's command shape, `git init` of a scratch directory followed by an empty `git -C <dir> commit`, `git -C <dir> worktree add` and `git -C <dir> worktree repair`, is not a violation.
- A `git init <dir>` in one command and `git -C <dir> commit` in a later command of the same fork is not a violation.
- `cd <dir> && git commit` after `git init <dir>` is not a violation.
- A commit inside a worktree added from a fork-created repository is not a violation.
- A commit inside a path outside every fork-created repository is a violation, including a sibling sharing the created directory's name prefix.
- `git push` from a fork-created repository, an invocation with `--git-dir` or `--work-tree`, a commit after a bare `cd`, and a commit after `git init` or `git init ..` are violations.
- A `cd` in an earlier command does not move a later command's directory.
- Every existing row of the mutating-git matrix keeps its verdict.
