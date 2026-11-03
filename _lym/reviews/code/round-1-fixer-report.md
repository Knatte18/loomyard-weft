```yaml
round: 1
fixed: [F1, F2, F3, F4]
disputed: []
deferred: []
```

## Changes

- **F1** — fixed as proposed. `internal/boardengine/decision.go` declares `DecisionRecordError`; `followDecisionRecords` returns the rename's record failure as that type.
  `Board.Rename` (`internal/boardengine/rename.go`) returns the saved result beside a `*DecisionRecordError` and an empty result for any other error; its doc comment says so.
  `internal/boardcli/cli.go` prints that error with `task` and `mentions` through `output.ErrFields`, and the rename `Long` says so.
  The `RenameOntoDirectoryReportsTheMoveAfterTheSave` row in `decision_test.go` now asserts the error type and the result.
  `board.go` and `doc.go` of `boardengine` describe the error form.
- **F2** — fixed as proposed, within the 2000-byte cap.
  The `decision add` Long restores that `--by` and `--amends-plan` apply to every entry, that the notice names the headings, and that two entries may not share a title or a PATTERN entry.
  It trims the second example, the sampler parenthetical and two phrases to stay under the cap.
  The upsert Long restores the meaning of `front`, `on_land` and `discussed` on one line, and the `--body-file` flag text is shortened.
  One wording of the F2 fix differs from the proposal: the notice is stated as "the notice names the headings", not "each live agent gets one notice", because the earlier paragraph already names the notification.
- **F3** — fixed as proposed. `decisionFileName` is exported as `DecisionFileName` (renamed, no wrapper, since nothing else uses the unexported name); the `get --record` message forms both file names through it.
- **F4** — fixed through the doc-comment option: `followDecisionRecords` says the orphan removal also runs for an unflagged entry. No behavior change.

Gate: `lym gate test ./internal/boardengine ./internal/boardcli ./internal/loomcli ./cmd/lym` passed tier0 after the first commit, and `./internal/boardengine ./internal/loomcli ./cmd/lym` after the second.
Commits: the four findings are in one commit, `0eb95002`, because F1 to F3 share `internal/boardcli/cli.go` and could not be split without interactive staging; the self-check fixes are the second.

## Disputed

None.

## Deferred

None.

## Self-check

- fix-consistency, LOW: `boardengine` `doc.go` and `board.go` did not describe `DecisionRecordError` — fixed as proposed.
- fix-proposal, LOW: the `add` Long dropped the Record-Write recipe qualifier — fixed as proposed, in other words ("on a Record-Write recipe").
- fix-proposal, LOW: "(restoring on a finding)" lost its object — fixed as proposed, the cap margin allowing it (content 1994 of 2000 bytes by count).
- fix-code-correctness: no findings. Its note on a missing CLI-level test of the `ErrFields` branch is not a finding under the rubric; the engine test covers the result.
- fix-affected-set: no findings.
- fix-proposal: no findings on F1, F3, F4.
