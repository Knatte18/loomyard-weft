# Decision record — burler-guard-readonly-git

## Goal

The burler fork git guard never fails a review round on a chain of read-only git commands, such as the one in issue #898:
`git branch -a --contains <sha>`, `git merge-base --is-ancestor <sha> HEAD` and `git show --stat <sha>`, piped through `head` and `grep` and joined by `&&`, `||` and `;`, with `2>/dev/null` redirections.

## Scope

- In: the shell splitting in `internal/burlerengine/cluster.go` that feeds the guard, so a redirection and its target are no words of a git invocation.
- In: a regression row in `TestFirstMutatingGitCommand` holding the issue's command line, and rows pinning redirections beside `git branch`.
- In: the guard's description in `internal/burlerengine/doc.go`.
- Out: the `git branch` list-mode classification, which commit 910ec75a already landed after the halt the issue reports.
- Out: the halt kind a true guard hit produces.

## Decisions

### D1 — A redirection and its target are dropped from the words of a segment

The shell splitter drops each unquoted `<` or `>` redirection operator, a file-descriptor number attached in front of it, an `&` attached after it and the target word that follows.
Rationale: today `git branch -a 2>/dev/null` reads `2>/dev/null` as a branch-name operand, so the guard counts a listing as branch creation.
Bound: only the redirection target leaves the words; a subcommand word anywhere else in the segment stays, so `git 2>/dev/null commit -m x` still counts as mutating.
A quoted or backslash-escaped `<` or `>` stays part of its word.

### D2 — The issue's command line is a regression row

`TestFirstMutatingGitCommand` gains a row holding the chain from issue #898, expected not mutating, beside rows for a redirected bare `git branch -a`, a `2>&1` redirect and a redirection that cannot hide a later mutating subcommand.
Rationale: the issue asks for a regression test with that command line.

### D3 — A guard hit stays a hard failure

A fork that the guard flags still fails the round as a producer hard failure; it is not downgraded to a retried attempt.
Rationale: on a true hit the fork may have changed the repository, and a retry would review on top of that change; the false positive is fixed at the classifier instead.

### D4 — The git branch listing fix is taken as landed

The branch listing rule in commit 910ec75a, landed after the reported halt, already passes `git branch -a --contains <sha>`; this task does not rework it.
Rationale: the board entry's first fix request is overtaken by that commit, and the remaining false positive in the reported command shape is the redirection.

## Constraints

- `PATTERN-test-speed`: the new rows join the existing `tier0` table, run through `lym gate test`.
- The guard stays lexical: no shell is run to classify a command.

## Auto-mode assumptions

- The issue's exact `<sha>` values are not recoverable, so the regression row uses an abbreviated sha in their place.

## Open risks

- Shell redirection forms the splitter does not model, such as a here-document body, keep their current reading.

## Acceptance criteria

- D1: `git branch -a 2>/dev/null` and `git branch 2>&1 | head` are not mutating, and `git 2>/dev/null commit -m x` is.
- D1: a quoted `">"` beside `git branch` still reads as an operand, so `git branch ">"` is mutating.
- D2: the issue #898 command chain is not mutating in `TestFirstMutatingGitCommand`.
- D3: `forkViolation` still returns an error for a mutating git command, unchanged by this task.
- D4: the existing `git branch -a --contains` rows keep passing unchanged.
