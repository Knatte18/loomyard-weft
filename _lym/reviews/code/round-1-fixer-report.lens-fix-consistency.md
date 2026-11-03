```yaml
lens: fix-consistency
outcome: reported
```

Findings (lens fix-consistency): one LOW, no BLOCKING or MEDIUM.

**LOW — `internal/boardengine/doc.go` and `board.go:70` were not updated for the new `DecisionRecordError` contract.**
- `board.go:70` still says only "The record step's error ... is returned last, with the write saved." It never says that a rename's record failure is now a `*DecisionRecordError`, or that the result accompanies it.
- `doc.go:292` describes the file following its entry through a rename. It says nothing about the failure form.
- A reader of the module doc has no pointer to the new error type or to the CLI's `{"ok":false,...,"task","mentions"}` envelope.
- This is a missing one-line twin edit, not a contradiction.

Checked and consistent:
- `Board.Rename` is called only by `boardcli/cli.go:241` outside tests, and that call is updated.
- The `decisionFileName` → `DecisionFileName` rename has no stale reference. The rest of the repo uses only the literal `decision-<slug>.md` in prose and comments.
- The rename `Long` text in `cli.go` ("beside "task" and "mentions"") matches the new `ErrFields` envelope.
- The `Rename` doc comment and the `followDecisionRecords` doc agree with the code.
- The `loomcli/doc.go` "Decision add --from" section already carries the contract that the help page now repeats in shortened form.

Calls made:
1. Ran grep over `internal` and `docs` for `.Rename(`, the literal file-name forms, `decisionFileName`, "record step" and `DecisionRecordError`. I used it to find twins and stale references that my context lacked.
