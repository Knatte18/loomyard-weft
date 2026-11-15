Task: check the fix's change set for agreement with its surroundings, through the fix-consistency lens.

Findings:

1. **LOW** — the doc comment of `TestUpsertTaskReportingRecordWarning` in `internal/boardengine/sketch_test.go:269-274` is stale. It lists the silent cases as "leaves the body alone", "restores the body the record was stored against" and "no stored record". F1 added two more silent rows, `BodyEditOfADoneDiscussedTaskIsSilent` and `BodyEditOfADiscussedTaskWithNoRecordFileIsSilent`. The comment is in the context unchanged, so neither row is described. The missing-file row also falls under "no stored record", but the done case is not covered by any line.

2. **LOW** — the F3 text in `plugins/ly/skills/board/SKILL.md` says "open entry". The surrounding text says the upsert is refused for a "done entry" only under `--record-file`, so the new sentence is consistent. It does not say that a body edit of a done entry is silent, which the new F1 row pins. This is a minor omission, not a contradiction.

Checked and consistent:
- The `UpsertTaskSiblings` rename in `doc.go:700`. The earlier repo grep (`.go` and `.md`) showed that line as its only hit.
- The F4 assertion. `stepUpsertRecordFile` is the only step that calls `--record-file` successfully, and its comment already says it pins that path.
- The table literals. All rows were widened to the new field order.

Further tool calls: none. Everything I needed was already in context.
