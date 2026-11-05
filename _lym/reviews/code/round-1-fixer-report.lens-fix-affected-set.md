```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set, task: re-run F1's search for every site that states or depends on the plain-promote rule and check the fix covers each one.

F1 was the only finding. It carried one search, over sites that state or rely on the plain-promote claim and over callers of `promoteTo`. The fix covers every code caller and every claim site, with two small stale texts left.

1. **LOW** — `internal/boardengine/sketch_test.go:200-201`, the `TestDiscussedPromotion` doc comment.
   - The comment says "a note whose record file is missing or that was never discussed promotes as any note does".
   - The new row `PlainPromoteOfASketchWithAMissingRecordFileTakesRaddle` (line 304) is a note with a missing record file that does not promote as any note does: it takes raddle.
   - The comment was not widened to say a label-carrying note with a missing record takes raddle.

2. **LOW** — `internal/boardengine/store.go:767`, the `promoteTo` doc comment.
   - The comment says "A nil recordStored reports no record stored."
   - `Store.Promote` (store.go:761) passes nil. A discussed note that carries the label now still takes raddle through that path, because the new condition at line 784 does not need `recordStored`.
   - The comment is accurate about the record but silent that the label alone suffices. The line 765 sentence covers the label, so this is a stale-looking sibling, not a wrong behaviour.

Sites checked and left consistent:
- `internal/boardengine/doc.go:322`, `internal/boardengine/doc.go:325` and `internal/boardengine/doc.go:327`.
- `internal/boardengine/labels.go:52` and `internal/boardengine/labels.go:67`: both claim the plain promote gives raddle, and that is now true for the labelled, record-missing case.
- `internal/boardengine/sketch.go:92` and `internal/boardengine/sketch.go:114`.
- `internal/boardengine/board.go:417`, whose only `promoteTo` caller (board.go:420) passes `b.recordStored`.
- `internal/boardcli/doc.go:80`, `internal/boardcli/cli.go:757` and `plugins/ly/skills/board/SKILL.md:136`.
- `internal/battencli/startguard.go:172`, which keys on its own `recordStored` and is unaffected.

Findings left unchecked: none.

Tool calls:
1. One `grep -rnE` over `*.go` and `*.md` for "plain promote", "which gives it recipe raddle", "record file is missing", `SketchPromoteCommand`, `recordStored` and `promoteTo(`. My context held only the files the fixer read, not the other claim sites or callers.
