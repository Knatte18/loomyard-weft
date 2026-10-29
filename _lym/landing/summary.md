# Burler fork git guard no longer reads a redirection as a git operand

Fixes #898.

The burler fork audit splits each fork Bash command into shell segments and judges every git invocation by its subcommand and arguments.
The splitter kept a redirection such as `2>/dev/null` as an ordinary word, so `git branch -a 2>/dev/null` read as `git branch` with a branch-name operand and failed the review round as a git-mutating command.

`splitShellSegments` in `internal/burlerengine/cluster.go` now drops each unquoted `<` or `>` redirection: the operator, a file-descriptor number attached in front of it, an `&` attached after it and the target word that follows.
A quoted or escaped `<` or `>` stays part of its word, and a subcommand word elsewhere in the segment is still judged, so `git 2>/dev/null commit -m x` still counts as mutating.
The guard's description in `internal/burlerengine/doc.go` names the rule.

`TestFirstMutatingGitCommand` gains rows for a redirected `git branch -a`, a `2>&1` redirect into a pipe, a spaced output redirect, a redirect before a mutating subcommand, a quoted `">"` operand, and the read-only chain from the issue: `git branch -a --contains`, `git merge-base --is-ancestor` and `git show --stat`, piped through `head` and `grep` and joined by `&&`, `||` and `;`.

## Design decisions

- The `git branch` list-mode classification the issue asks for had already landed in commit 910ec75a, after the halt it reports; this change leaves it as it is and fixes the redirection false positive that remained in that command shape.
- A guard hit stays a producer hard failure and is not downgraded to a retried attempt: on a true hit the fork may have changed the repository, and a retry would review on top of that change.
