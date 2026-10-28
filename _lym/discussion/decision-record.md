# Decision record — fork-audit-branch-list

## Goal

A burler review fork that lists branches, as in `git branch -a --contains 71a46b11`, no longer fails the review round as a git-mutating fork.

## Scope

- `internal/burlerengine/cluster.go`: `git branch` is judged by its arguments, as `git tag` already is, and counts as mutating only when it may create, delete, rename, copy or move a branch or change its upstream or description.
- `internal/burlerengine/cluster_test.go`: rows for the read-only and the writing forms.
- `internal/burlerengine/doc.go`: the fork-audit paragraph names the `git branch` list exception beside the `git tag` one.

## Decisions

### Decision: judge git branch by an allow-list of read-only options

`git branch` reads only when every option it carries is a known read-only option and it either has no operand or carries an option that puts it in list mode.
The read-only options are `-a`/`--all`, `-r`/`--remotes`, `-l`/`--list`, `-v`/`-vv`/`--verbose`, `-i`/`--ignore-case`, `--contains`, `--no-contains`, `--merged`, `--no-merged`, `--points-at`, `--show-current`, `--format`, `--sort`, `--column`, `--no-column`, `--color`, `--no-color`, `--abbrev`, `--no-abbrev` and `--omit-empty`;
the list-mode ones are `-l`/`--list`, `-v`/`--verbose`, `--contains`, `--no-contains`, `--merged`, `--no-merged`, `--points-at` and `--show-current`.
Any other option, including every writing one and any option this list does not know, keeps the command mutating.
Rationale: an allow-list fails loud on an option the audit has not classified, which suits a guard whose miss lets a fork change the reviewed branch.
Bound: the exception lets through only `git branch` invocations git itself runs as a listing or `--show-current`; a branch name operand without a list-mode option still counts, as `git branch -a foo` creates `foo`.

### Decision: -a, -r, --format and --sort read only without an operand

These options do not put `git branch` into list mode, so with a branch-name operand git creates that branch.
They pass with no operand, and beside an operand only when a list-mode option is present too.
Rationale: this follows git's own mode selection, so the audit never passes a command that creates a branch.

### Decision: short-option clusters are read letter by letter

A cluster such as `-av` or `-vv` passes when every letter is a read-only short option, and any other letter keeps the command mutating.
Rationale: `git branch -avv` is a common listing form, and reading each letter keeps a writing letter such as `-D` in a cluster caught.

### Decision: valued options skip their value

`--format`, `--sort` and `--points-at` given as a separate word take the following word as their value, so a value such as `-committerdate` is never read as an option and a value is never read as an operand.

## Constraints

- PATTERN-test-speed: the new cases are rows in `TestFirstMutatingGitCommand`, the table that already covers the classification.
- The audit's other rules, the fork-created-repository exception and the help-request exception, apply to `git branch` unchanged.

## Auto-mode assumptions

- A darn run holds no interview; the scope is the board entry's expected behavior, refined against git's own mode selection as the Decisions state.

## Open risks

- A read-only option git adds later counts as mutating until the allow-list names it; the failure is loud and the fix is one map entry.

## Acceptance criteria

- `git branch`, `git branch -a`, `git branch -r`, `git branch -avv`, `git branch -a --contains <commit>`, `git branch --contains=<commit>`, `git branch -l 'feat*'`, `git branch --list feat*`, `git branch --merged main`, `git branch --no-merged main`, `git branch --points-at HEAD`, `git branch --show-current` and `git branch --sort -committerdate` are not mutating.
- `git branch new`, `git branch -d old`, `git branch -D old`, `git branch -m a b`, `git branch -c a b`, `git branch -u origin/main`, `git branch --set-upstream-to=origin/main`, `git branch --unset-upstream`, `git branch -f main HEAD`, `git branch -a foo`, `git branch -l -d old`, `git branch -aD old` and `git branch -- new` are mutating.
- The existing `git tag` and other rows of `TestFirstMutatingGitCommand` keep passing.
