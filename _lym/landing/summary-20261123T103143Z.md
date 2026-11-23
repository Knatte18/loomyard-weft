# Refuse a CLI error envelope as a board body or decision-record Goal

A darn run's decision record carried `{"error":"invalid json: invalid character 'h' ...","ok":false}` as its `## Goal`, because the board entry's body itself held that text.
The board history shows how it got there: the entry was a merged note whose body read `## From <slug>` followed by an error envelope, once per merged note.
`lym board get <slug>` with a bare slug failed with `invalid json`, since a lookup payload took only JSON or a bare handle, and the orch role docs teach the bare-slug form.
The failed `get --body` output went into the file that `upsert --body-file` then stored, and Record-Write rendered the body into the record unchecked.

## What changes

- `output.ErrorEnvelopeLine` finds a line of text that is a whole error envelope: a JSON object with `"ok": false` and a string `"error"`.
  An envelope inside a prose line, a success envelope and plain text pass.
- Record-Write halts Stuck when the entry body it is about to render holds such a line, writing and committing nothing.
  The reason names the envelope, and its way forward is to end the run with `lym batten cancel`, rewrite the body with `lym board upsert --body-file`, and start the run again.
  Cancelling first is needed because the run holds the entry's scope.
- `lym board upsert --body-file` and `merge --body-file` refuse a file holding an error envelope, naming it, so the failed capture is caught at the board write.
- Every `lym board` lookup verb reads a bare slug, such as `lym board get my-task`, as `{"slug":"my-task"}`.
  A bare payload qualifies when it starts with a letter and holds only letters, digits, `-`, `_` and `.`.
  A bare handle such as `B4` still reads as a handle first.

## Design decisions

- The detector lives in `internal/output`, which writes the envelope, so one package owns both sides of the format.
- Record-Write checks only a body it renders, never a stored decision record, which a Discussion approved.
- A payload body given inline in JSON is not checked: the guard targets the file and stdin route that captured command output takes.

## Tests

- `TestErrorEnvelopeLine` pins the shapes the detector accepts and passes.
- `TestRecordWrite_Call/ErrorEnvelopeBodyIsStuckNamingTheEnvelope` reproduces the reported body and asserts Stuck with nothing written.
- `TestApplyBodyFile` gains a row refusing a file holding an envelope.
- The board CLI table gains `get_by_bare_slug`, which failed with `invalid json` before the change.

## Merge danger

**Door:** two-way
**Blast radius:** board
