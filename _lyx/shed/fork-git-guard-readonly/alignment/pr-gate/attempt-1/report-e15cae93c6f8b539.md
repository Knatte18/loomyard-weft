---
verdict: APPROVED
---

## Scope against the board entry

The board entry `fork-git-guard-readonly` (issues #750, #751) asks for three things.
The fork's git-mutation check must classify by the invoked git subcommand, the first non-option word after git and its global options such as `-C <dir>`, and not by mutating verbs named in arguments.
`help`, `--help`, `-h` and man-page lookups must count as read-only.
There must be a test case for `git help <mutating-verb>`.

The decision record has no `Added after Discussion` entries, so the board body is the measure on its own.

The diff of PR 752 at `5c50abf7b` against `main` covers each point:

- `internal/burlerengine/cluster.go` replaces `mutatingGitPattern` with `commandRunsMutatingGit`.
  It splits the command into segments, finds the git invocation, skips the global options and their values, and checks only the subcommand word against the same mutating set.
- `git help <verb>` passes because `help` is not in the mutating set.
  `-h` or `--help` among the global options or directly after the subcommand also passes.
- `internal/burlerengine/cluster_test.go` renames the existing table and extends it.
  The new rows include the exact command from the issue, `git help -m cherry-pick ... | col -b | grep ... | head -30`, plus `git log --grep merge`, `git show tag`, `git --help commit`, `git commit --help` and `git rebase -h`.
- `internal/burlerengine/doc.go` states the subcommand rule in the fork-audit paragraph.

The diff does nothing the entry does not call for.
It leaves the mutating set and the other audit checks unchanged.

## Open risks

The record carries one open risk: a quoted argument holding a separator character splits a segment early.
The final tokenizer handles quotes, so the risk is now narrower than the record states.
The risk names no parent correction, so it is no reason to escalate.

## Overlap

`lyx loom peers` lists no parent commits (`own_row` 21).

- `ubuntu-2404-support` (row 5): no overlap.
  Its run hit this bug, but its decision record does not touch `burlerengine`, the fork audit or the mutating-git check, and it lists no changed paths.
- `hub-workspaces` (row 9): no overlap.
  Its decision record covers workspace files and hub containment, and it lists no changed paths.

No board entry other than this one covers the fork git-mutation guard.
Since nothing overlaps, no landing order applies.
This run makes no other board entry obsolete.
