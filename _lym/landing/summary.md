# Refuse a CLI error envelope as a board body; accept a bare slug

A Darn-Write session found a decision record whose `## Goal` held `{"error":"invalid json: invalid character 'h' ...","ok":false}` instead of the task's text.
Nothing refused it before Discussion, so the agent rebuilt the scope from linked issues.

The upstream cause was `lym board get my-task`: a bare slug was not accepted as a payload, only a bare handle such as `B4`.
The verb printed an `invalid json` envelope, and a caller captured that output into the file it passed as the body.

Closes #181

## What changes

- `output.ErrorEnvelopeLine` finds a line that is a whole error envelope (`"ok":false` plus a string `"error"`).
  A line inside a closed fenced code block is skipped, so a body that quotes an envelope in a block passes.
  A fence left open to the end of the text hides nothing.
- `lym board upsert --body-file` and `lym board merge --body-file` refuse a file holding such a line.
  The refusal names the failed verb as the cause.
- The Record-Write row reports `Stuck` when the board entry's body holds an envelope, and writes no record.
  The reason gives a way forward for a beam project (fix the body, `lym loom resume`)
  and for any other run (cancel with `--remove`, fix the body, release the slug, run again).
- `lym board get`, and every verb taking one entry's selector, accepts a bare slug such as `my-task` as its whole payload.
  This removes the failure that produced the envelope.
  The help text, `payloadLegend`, the `boardcli` package doc and the board skill document it.

## Design decisions

- A bare slug is recognised by shape: a letter first, then letters, digits, `-`, `_` and `.`.
  No JSON object has that shape, so the reading never shadows a payload.
- The envelope check is on the line, not on a JSON parse of the whole body, so an envelope followed by other text is still caught.
- The check lives in `internal/output`, beside the envelope writers, so both `boardcli` and `loomshed` share one definition.
- Record-Write refuses by returning `Stuck` with a way forward instead of failing the run, matching its existing no-entry case.

## To check by hand

- `errorenvelopeline-matches-fenced-quote`: `ErrorEnvelopeLine` in `internal/output/output.go` skips closed fences only.
  Check that a body quoting an envelope inside an unclosed fence being refused is the behaviour wanted.
- `recordwrite-envelope-reason-way-forward`: `errorEnvelopeBodyReason` in `internal/loomshed/recordwrite.go` names two recovery paths.
  Check each command in it against a real run, since none was run by hand.

## Merge danger

**Door:** two-way
**Blast radius:** board
