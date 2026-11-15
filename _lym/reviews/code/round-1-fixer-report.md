# Fixer report — code review, round 1

A follow-up commit repaired the gate's comment findings:
the `readEdits` doc comment in `edits.go`, the `TestArbiterEnvelopeFields` comment and the `fakeWorktree` comment now break at sentence boundaries, one sentence per line, under 120 characters.
No code changed.

All other changes are in one commit, `b2837efe`, because the findings share files (`edits.go`, `call.go`, `call_test.go`).
Each finding is not committed alone as the fix rules ask.
The gate (`lym gate test ./internal/shedarbiter ./internal/loomcli ./internal/loomengine`) passed: 2163 passed, 0 failed.

## Changes

- **F1 (BLOCKING), fixed with a variation of the proposal.**
  `WorktreeSeam` gains `Changes`, wired in `arbiterWorktreeSeam` through `Fabric.CodeWorktreeChanges`.
  The call records its result in the attempt as `ArbiterAttempt.Changes` at spawn.
  `readEdits` sets "the arbiter left uncommitted changes in the worktree" when the current set differs from the recorded one, whatever the order.
  The proposal kept the set in the spawn snapshot.
  I put it in `ArbiterAttempt` because `sheddriver.WorktreeSnapshot` is shared with the tackler, which the finding does not cover.
  A set that includes tracked paths too is harmless: `Dirty` already flags those.
  The `Snapshot` doc comment now says it never lists an untracked file.
  Rows added to `TestCall_Edits`: a new untracked file (hand-on), an untracked file found at spawn and left alone (resume), and a `Changes` read failure.
- **F3 (MEDIUM), fixed as proposed.**
  `fakeWorktree` has separate errors for the spawn read, the post-arbiter `Snapshot`, `Changes`, `IsAncestor` and `CommitLog`.
  Four rows cover each post-arbiter failure, including the detached HEAD.
- **F4 (LOW), fixed as proposed, with one addition.**
  Both commits-could-not-be-read problems read "HEAD moved from X to Y and the arbiter's commits could not be read: err".
  `conclude` now passes the whole `edits` to `handOnWith` on a resume with a problem, not just its commits.
  Without that, the problem reached only the reason, not the question, and the rows failed.
- **F5 (LOW), fixed as proposed.**
  Two rows: a resume over uncommitted changes after commits, listing them, and an agent's hand-on over a dirty worktree, gaining the worktree note.
- **F6 (LOW), fixed as proposed.**
  The row "a resume after commits with a goto lists them" also calls `Seam(...).Judge` after the settle and asserts the outcome's commits.
- **F7 (LOW), fixed as proposed.**
  `TestArbiterEnvelopeFields` in `internal/loomcli/arbiterprompt_test.go`.
- **F8 (LOW), fixed as proposed.**
  The `//testtiming:keep` line sits in `TestPrompt`'s doc comment, separated by a blank `//` line.
- **F9 (LOW), fixed as proposed.**
  A bullet in `loom-fragment-arbiter-fix.md` limits the changing git commands to `git add` and `git commit` on the branch found.
  The template's "Push, amend, rebase, reset or switch a branch" line stays.
  The untracked-file case from F1 also appears in the "leave no uncommitted change" bullet.
- **F10 (LOW), fixed as proposed.**
  `internal/loomcli/doc.go` `# Arbiter` section gains the worktree seam, the attempt record and the commits sentences, and names the new uncommitted-paths record.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings; F3–F9 were left unchecked as not searchable sets.

## Disputed

None.

## Deferred with reason

None.
`F2` is a `decision`-class finding held by the parent, so this report leaves it out.

## Rulings

None reached me this round.
