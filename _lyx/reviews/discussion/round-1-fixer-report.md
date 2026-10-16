# Fixer report — discussion, round 1

## Changes

All edits are in `_lyx/discussion/decision-record.md`.

- **F1** (fixed as proposed): D4's close-as-noise bullet now allows a close only for an issue that adds nothing to the issue or entry it duplicates, or one with no actionable content; a duplicate that adds anything is folded, or becomes a new note when the entry is run-held.
- **F2** (fixed as proposed): D1 and the acceptance criterion give the empty inbox as `{"issues":[]}`.
  The premise matches `intakeList`, which prints through `output.Ok(out, map[string]any{"issues": views})`.
- **F3** (fixed as proposed, with one wording adaptation): D4's re-run runs the `close` payload once, accepts a second pointer comment when the first posted, and gives the issue a `failed` line with the error and the payload on a second failure.
  D3 gains a bullet handling a `failed` line as a lyx bug under Where a finding goes.
  The bullet names the `close` payload only "for a failed close", since a `failed` line can also come from a failed import, which has no payload.
  The premise matches `selfreportengine.CommentAndClose`, whose error says a rerun posts a second comment.
- **F4** (fixed as proposed): D4's Bound confines the upsert to the labels of an entry the fork just folded into, keeping every label and adding the issue's module labels.
- **F5** (fixed as proposed): the last acceptance criterion now requires `internal/boardcli/doc.go` to state D1's list output, the `--full` flag and D2's import output.

## Disputed

None.

## Deferred

None.
