# Fork git guard classifies by subcommand, not by argument

A review fork that read a man page with `git help -m cherry-pick 2>/dev/null | col -b | grep ... | head -30` hard-failed a Burler round as a state-mutating git command.
`mutatingGitPattern` in `internal/burlerengine/cluster.go` matched a mutating verb anywhere among git's later words, so `git help cherry-pick`, `git log --grep merge` and `git show tag` were refused too.
The audit now refuses a Bash command only when the git subcommand it invokes mutates.

Closes #750
Closes #751

## What changes

- `mutatingGitPattern` is replaced by `commandRunsMutatingGit`, a small tokenizing function.
  It splits the command into segments at unquoted `;`, `&`, `|` and newlines.
  A segment counts when its first word is `git`, bare or path-prefixed, and its subcommand is in the unchanged `mutatingGitSubcommands` set.
- The subcommand is the first non-option word after git's global options.
  `-C`, `-c`, `--git-dir`, `--work-tree`, `--namespace`, `--config-env` and `--attr-source` take a separate value word, which is skipped.
- `-h` or `--help` among the global options or directly after the subcommand marks a usage request and passes, as does `git help <verb>`, since `help` is not a mutating subcommand.
  Further along, `-h` is an operand or option value, so `git commit -m -h` still counts as mutating.
- The `forkViolation` call site and the error message are unchanged.
- `internal/burlerengine/doc.go` states the subcommand rule in the fork-audit paragraph.
- `TestMutatingGitPattern` in `cluster_test.go` is extended and renamed for the new function rather than duplicated.
  It covers the issue's command, read-only subcommands naming a mutating verb, help escapes, global-option value skips, quoting, escapes, comments and continued lines, and the mutating shapes (path-prefixed git, `&&` and `;` chains, a commit on a later line).

## Design decisions

- **Plain Go over a regex.**
  Skipping value-taking global options and finding the first non-option word is a positional rule a regex states poorly.
- **The tokenizer is quote-aware.**
  A quoted run is one word, so a separator inside quotes does not split a segment, and a quoted `"git"` argument is not mistaken for a command.
  Backslash escapes, `#` comments and backslash-newline continuations follow shell rules closely enough for the guard.
- **A newline separates commands like `;`.**
  The old pattern missed a mutating command on a later line; the new classifier catches it.
- **The set of mutating subcommands is unchanged.**
  Parent-transcript checks elsewhere are untouched.
- **Known gaps, unchanged from before.**
  A command inside a subshell `(...)` or behind an environment assignment (`FOO=1 git commit`) is not classified.

## To check by hand

Nothing outside the unit table was run against a live fork.
Re-running a Burler review round whose fork reads a git man page would confirm the end-to-end path.
