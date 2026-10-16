# Card 2 — intake import prints a short result

**Edit:**
- `internal/boardcli#intakeCommand`
- `internal/boardcli#intakeImport`
- `internal/boardcli#TestIntakeImport_NewNoteCommentsAndCloses`
- `internal/boardcli#TestIntakeImport_FoldIntoEntry`
- `internal/boardcli/doc.go`

**Rename:**
- `internal/boardcli#TestIntakeImport_CloseFailureReportsEntryAndClosePayload` -> `internal/boardcli#TestIntakeImport_CloseFailureReportsShortResultAndClosePayload`

**Uses:**
- `internal/boardengine#ImportResult`
- `internal/boardengine#Board.GetTask`
- `internal/boardcli#newFakeInbox`
- `internal/boardcli#newIntakeBoard`
- `internal/boardcli#runHandler`
- `internal/boardcli#readBoard`
- `internal/boardcli#TestIntakeImport_RecordedIssueIsNoOp`

**Intent:**
D2: a successful `intake import` prints where the issue went and with which labels, never the entry.

- `intakeImport` builds one result map, `slug` (`result.Entry.Slug`), `issue` (the issue number), `labels` (`result.Entry.Labels`, never empty, since the board refuses an entry with no type label) and `dropped` (today's nil-to-empty `dropped`).
  A new note and a fold print it the same way through `output.Ok`; for a fold `slug` is the entry folded into and `labels` its labels after the fold.
  When `selfreportengine.CommentAndClose` fails after the board write, `output.ErrFields` carries the same four fields plus today's `close` payload, in place of `entry`.
  The already-recorded no-op keeps printing `recorded` and touches nothing on GitHub, which `TestIntakeImport_RecordedIssueIsNoOp` keeps covering unchanged.
- `intakeCommand`'s `import` command's `Long` replaces "the error carries the written entry and the "close" payload to run" with a statement that a success prints slug, issue, labels and dropped, a fold naming the entry folded into, and that a failed comment or close puts those four fields and the "close" payload to run in the error.
  A grep for "the written entry" finds that `Long` alone; no test asserts it.
- The envelope is decoded into `map[string]any`, so each `issue` assertion below compares against a `float64` (`7.0`, `8.0`, `16.0`), as today's close-payload check does.
- `TestIntakeImport_NewNoteCommentsAndCloses` asserts the envelope's `slug` `from-inbox`, `issue` 7, `labels` `[bug]` and `dropped` `[stray]`, and that it has no `entry` and no `body` key.
  Its body assertion, which pins the dropped whole-body pointer and the kept quoted one, moves onto the stored entry read through `b.GetTask("from-inbox")`, together with its `note` kind.
- `TestIntakeImport_FoldIntoEntry` gives issue 8 the module label `undecided` and asserts the envelope's `slug` `target`, `issue` 8, `labels` `[bug undecided]`, `dropped` `[]` and no `entry` key.
  Its `issues` and body-tail assertions move onto the stored entry read through `b.GetTask("target")`.
- `TestIntakeImport_CloseFailureReportsEntryAndClosePayload` is renamed `TestIntakeImport_CloseFailureReportsShortResultAndClosePayload`, since it now pins that the entry is left out, and asserts the error envelope's `slug` `half-done`, `issue` 16, `labels` `[bug]`, `dropped` `[]` and no `entry` key, beside today's `close` payload and board-written assertions.
- `doc.go` gains a paragraph stating that `intake import` prints `{"slug":…,"issue":…,"labels":[…],"dropped":[…]}`, a fold with the slug of the entry folded into, that a failed comment or close after the board write carries the same four fields and the `close` payload in the error envelope, and that an already-recorded issue prints `recorded`.

**ImpactSummary:** `intakeImport` keeps its signature and has one production caller; only the three listed tests read its output, and no other package parses it.

**Commit:** 2: intake import prints a short result
