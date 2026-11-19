---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
---

## Changes

- F1, commit 7ab62ffd: `contracts/orchrole/role-intake.md` now makes the comment-to-note split conditional on the fork's own import of that issue having written the board, a success or an error carrying a `close` payload, and skips it when the import prints `recorded`.
- F2, commit 69e45917: `intakeList` in `internal/boardcli/intake.go` wraps a failed comment fetch as `issue #<n>: <error>`.
- F3, commit 69e45917: `TestIntakeList_ExcludesPullRequestsRecordedAndClosedAcrossPages` gained a `wantErr` field.
  The `CommentListFailure` row sets it to `issue #1: github issue comment list failed`, and the failure branch asserts `strings.Contains(msg, tt.wantErr)`.
  F3's assertion depends on F2's message, so both share one commit.

`lym gate test ./internal/boardcli` passed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal, LOW, F1 loses a split note when a fork dies after its import wrote the board: not changed.
  The window is the inherent price of the reviewer's condition, which trades a duplicate note for a possible missing one;
  closing it means the operator choosing a different rule for the split.

## Disputed

None.

## Deferred

None.

## Rulings

None.
