# Fixer-report — round 1

## Changes

- F1: fixed as proposed. Row `git 2>&1 commit -m x` added to `TestFirstMutatingGitCommand` (`internal/burlerengine/cluster_test.go`).
- F2: fixed as proposed. `splitShellSegments` (`internal/burlerengine/cluster.go`) gains a `wordQuoted` flag, set in the quote-open and backslash-escape cases and cleared in `endWord`; the fd check applies only when it is false. Row `git branch "2">x` added.
- F3: fixed as proposed. Rows added for a redirect with no target before a separator, a detached digit word, and an escaped redirect character.
- Self-check fix: the `splitShellSegments` doc comment now says "unquoted file-descriptor number".

Commits: `ea4d39f1` (F1, F2, F3 together, one commit because the edits share two files) and a follow-up for the comment wording.
`lym gate test ./internal/burlerengine` is green.

## Self-check

- fix-consistency, LOW: doc comment named any fd number; fixed as proposed (wording "unquoted").
- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
