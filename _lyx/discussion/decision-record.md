# Decision record — fork-git-guard-read-only-tag

## Goal

The fork git-mutation audit in `internal/burlerengine/cluster.go` stops refusing read-only `git tag` forms, such as `git tag --points-at HEAD`, while a `git tag` that creates or deletes a tag stays refused.

## Scope

- `commandRunsMutatingGit` in `internal/burlerengine/cluster.go` classifies a `git tag` invocation by its arguments, not by the subcommand word alone.
- Every other subcommand keeps the subcommand-only rule the guard was given on purpose.
- The table test `TestCommandRunsMutatingGit` gains rows for the issue's command line and the read-only and mutating `git tag` forms.
- The fork-audit paragraph in `internal/burlerengine/doc.go` states the `git tag` exception.

## Decisions

### A list-mode flag makes `git tag` read-only

Decision: a `git tag` invocation is read-only when its arguments, up to a `--`, hold a list-mode flag and no tag-writing flag, or when it has no arguments at all.
The list-mode flags are `-l`, `--list`, `--points-at`, `--contains`, `--no-contains`, `--merged`, `--no-merged` and `-n`, the long ones also in their `=value` form and `-n` also with its count attached, as in `-n5`.
The tag-writing flags are `-d`, `--delete`, `-a`, `--annotate`, `-s`, `--sign`, `-u`, `--local-user`, `-f`, `--force`, `-m`, `--message`, `-F`, `--file`, `-e` and `--edit`, the long ones also in their `=value` form.
Rationale: git documents each listed flag as implying list mode, in which non-option words are patterns, never a tag to create, and bare `git tag` lists every tag.
A tag-writing flag beside a list-mode flag still counts as mutating, so a mixed or ambiguous command stays refused.
Bound: this weakens the guard for `git tag` only.
It lets through a `git tag` command holding one of the named list-mode flags and none of the named tag-writing flags, and bare `git tag`;
any other `git tag`, including one with a combined short-flag cluster such as `-ln`, stays mutating.

### Other multi-purpose subcommands stay subcommand-only

Decision: `branch` and `stash` keep counting as mutating whatever their arguments, and `remote` and `config` stay outside the mutating set.
Rationale: the operator decision on the board entry scopes the change to `git tag` and keeps the subcommand-only rule otherwise.

### The halt message stays as it is

Decision: the fork-violation message and the twill fixer's way forward are unchanged.
Rationale: with the false positive gone, a refused fork command writes a commit, ref or working-tree change, so "inspect the target's working-tree changes, revert what the fork wrote" still names the recovery.

## Constraints

- `PATTERN-refusal-way-forward`: the refusal's way forward stays reachable by an existing test.
- `PATTERN-comment-line-breaks`: Go comments are one sentence per line.
- `PATTERN-test-economy`: the new cases are rows in `TestCommandRunsMutatingGit`, not a new test.

## Auto-mode assumptions

- The board body's longer list of read-only forms (`--merged`, `--no-merged`, `-n`) falls inside the operator decision's intent, since git documents each as implying list mode.
- `--no-contains` is included on the same ground.

## Open risks

- `git branch` and `git stash` have the same flaw: `git branch --list` or `git stash list` in a fork still halts the run.
  The operator decision leaves them out of this task.

## Acceptance criteria

- The issue's command line, a `sed -n` read, a `git log -1` and a `git tag --points-at HEAD` joined by `;`, is not mutating.
- `git tag` with `-l`, `--list`, `--points-at`, `--contains`, `--no-contains`, `--merged`, `--no-merged` or `-n` is not mutating, in both separate-value and `=value` forms where those exist, and bare `git tag` is not mutating.
- `git tag v1.0`, `git tag -d v1`, `git tag -a v1 -m msg`, `git tag -s v1`, `git tag -f v1` and `git tag -l -d v1` are mutating.
- A `--list` after `--`, or as the value of `-m`, does not make a creating `git tag` read-only.
- Every existing row of `TestCommandRunsMutatingGit` keeps its result.
