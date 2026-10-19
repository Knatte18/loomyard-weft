# Decision record — fork git-mutation guard classifies by subcommand

## Goal

Burler's fork audit refuses a fork's Bash command as state-mutating only when the git subcommand it invokes mutates, so a read-only lookup such as `git help -m cherry-pick | col -b | grep ...` no longer hard-fails a review round.

## Scope

- `internal/burlerengine/cluster.go`: replace `mutatingGitPattern`, which matches a mutating verb anywhere among git's later words, with a classifier that reads the subcommand word.
- `internal/burlerengine/cluster_test.go`: the classifier's table gains the read-only shapes the issues name.
- `internal/burlerengine/doc.go`: the fork-audit paragraph states the subcommand rule.
- Out of scope: the set of mutating subcommands itself, and the parent-transcript checks elsewhere.

## Decisions

### The subcommand word alone decides

Decision: split the command into segments at `;`, `&`, `|` and newlines; a segment whose first word is `git`, bare or path-prefixed, mutates when its subcommand, the first non-option word after git's global options, is in the existing mutating set.
Global options taking a separate value (`-C`, `-c`, `--git-dir`, `--work-tree`, `--namespace`, `--config-env`) skip their value word.
Rationale: issue #751's expected behavior; `git help cherry-pick`, `git log --grep merge` and `git show tag` name a mutating verb only as an argument.

### Help requests are read-only

Decision: a segment holding `-h` or `--help` before or after a mutating subcommand passes, as does `git help <verb>`, since `help` is not in the mutating set.
Rationale: issue #750 names `help`, `--help` and `-h` as read-only; each prints usage or a man page and exits without touching the repository.
Bound: the escape lets through only a segment carrying the literal word `-h` or `--help`; git's builtins print usage and exit on either, so no mutating subcommand runs.

### Plain Go over a regex

Decision: the classifier is a small tokenizing function, not a regular expression.
Rationale: skipping value-taking global options and finding the first non-option word is a positional rule a regex states poorly.

## Constraints

- `PATTERN-test-economy`: the existing `TestMutatingGitPattern` table is extended and renamed for the new function, not duplicated.
- `PATTERN-comment-line-breaks`: comments one sentence per line.

## Auto-mode assumptions

- A newline separates commands as `;` does; the old pattern missed a mutating command on a later line, and treating it as a separate segment is the stricter reading.
- Commands wrapped in a subshell `(...)` or prefixed with an environment assignment stay unclassified, as before.

## Open risks

- A quoted argument holding a separator character splits a segment early; the split can only expose an extra segment beginning with a non-git word, so it never refuses a read-only command it previously passed.

## Acceptance criteria

- `git help -m cherry-pick 2>/dev/null | col -b | grep -n -A14 "^ *--empty" | head -30` is not mutating.
- `git log --grep merge`, `git show tag`, `git -C dir log`, `git --help commit` and `git commit --help` are not mutating.
- Every listed mutating subcommand, a path-prefixed git, a chained `&&` or `;` git call, `git -C dir commit`, `git -c key=value commit` and a commit on a later line are mutating.
