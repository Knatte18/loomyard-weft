# Burler fork git guard: ignore shell redirections when judging git

The burler fork guard no longer fails a review round on a read-only git chain that carries a redirection such as `2>/dev/null`.
The command from the report, `git branch -a --contains <sha> 2>/dev/null | head -20 && git merge-base --is-ancestor <sha> HEAD ... ; git show --stat <sha> 2>/dev/null | head -40 | grep -v "^$"`, now passes.

Closes #898

## Cause

`splitShellSegments` in `internal/burlerengine/cluster.go` cut a command into words without modelling redirections.
`git branch -a 2>/dev/null` therefore gave `git branch` the operand `2>/dev/null`, which the guard read as a branch name being created, and the fork was flagged as mutating.
The `git branch` list-mode rule that already landed passes `--contains` and the other listing flags; the redirection was the remaining false positive in the reported command shape.

## Change

- `splitShellSegments` drops an unquoted `<` or `>` redirection from the words of a segment: the operator, a file-descriptor number attached before it, an `&` attached after it, and the target word that follows.
- Only the target leaves the words.
  A subcommand word elsewhere in the segment stays, so `git 2>/dev/null commit -m x` and `git 2>&1 commit -m x` are still mutating.
- A quoted or backslash-escaped `<` or `>` stays in its word, so `git branch ">"` and `git branch \>` are still mutating.
  A quoted digit before a redirect, as in `git branch "2">x`, and a digit detached from it, as in `git branch 2 > x`, are operands, not file descriptors.
- A redirection with no target before a separator drops no word of the next segment.
- The guard stays lexical; no shell classifies a command.
- `internal/burlerengine/doc.go` states the redirection rule in the guard's description.

## Decisions

- A guard hit stays a producer hard failure.
  The issue suggests downgrading it to a retried attempt, but on a true hit the fork may have changed the repository and a retry would review on top of that change.
  The false positive is fixed in the classifier, and `forkViolation` is unchanged.
- The `git branch` list-mode classification is not reworked here.
- Redirection forms the splitter does not model keep their current reading: here-document bodies, `>|`, `&>` and process substitution.

## Tests

`TestFirstMutatingGitCommand` gains rows for the chain from the issue (with an abbreviated sha, since the original values are not recoverable), redirected `git branch` listings, and the mutating cases above.
The rows join the existing `tier0` table.
No test run is recorded in this description.

## Check by hand

- `internal/burlerengine/cluster_test.go:381`: the `&` skip after a redirection operator is pinned only by the row `git 2>&1 commit -m x`; confirm that row fails when the skip in `splitShellSegments` is removed.
- `internal/burlerengine/cluster.go:412`: the file-descriptor test must treat a quoted or escaped digit as an operand; confirm `git branch "2">x` is mutating.
- `internal/burlerengine/cluster_test.go:380`: confirm the rows for a redirect with no target before a separator, a detached digit operand and a backslash-escaped redirect character each fail without the behaviour they pin.
