---
round: 1
focus:
  - "internal/burlerengine/cluster.go `gitBranchReadsOnly` and its list-mode maps: check each option marked as list mode (`-v`/`--verbose` in particular) against git's own mode selection in `git branch`; does `git branch -v foo` or `git branch --verbose foo` list or create `foo`, and does the diff then let a branch-creating command pass?"
  - "`gitBranchReadsOnly` operand and value handling: walk the board entry's expected-form list and the decision record's acceptance criteria through the loop (`--contains <commit>` as a separate word, `--contains=<commit>`, `--sort -committerdate`, `-l 'feat*'`, `-- new`, `-aD old`, `-l -d old`, `-a foo`); does any listed read-only form come out mutating, or any listed writing form come out read-only?"
  - "Option coverage against the board entry's expected list: every option the entry names as read-only (including `-v`, `--show-current`, `--format`, `--sort`) is accepted, and every writing option (`-d`, `-D`, `-m`, `-M`, `-c`, `-C`, `-u`, `--set-upstream-to`, `--unset-upstream`, `-f`, `--edit-description`) keeps the command mutating, short clusters and `--name=value` forms included."
  - "Interaction with the rest of `gitInvocationMutates`: the fork-created-repository exception, the help-request exception and the `git tag` branch; does the new `case \"branch\"` leave them unchanged, and is `branch` still in `mutatingGitSubcommands` for the writing forms?"
  - "internal/burlerengine/cluster_test.go: do the new `TestFirstMutatingGitCommand` rows cover each acceptance criterion in the decision record, and do any duplicate an existing row's coverage?"
  - "internal/burlerengine/doc.go and the comment on `firstMutatingGitCommand`: do they name the `git branch` list exception accurately, matching the code's actual rule?"
---

Seed focus for the single-commit diff over `96a0e2bd`.
