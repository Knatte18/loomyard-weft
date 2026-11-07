```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: I checked whether the change set agrees with its siblings and twins. I found two LOW findings and no BLOCKING or MEDIUM ones.

- **LOW, `internal/heddleengine/overview_test.go`, doc comment of `TestRenderOverview`:** the comment is now stale.
  - It still says "It uses a store holding a text entry and a JSON-object entry."
  - The fix added two more entries to that store, `fenced-rules` (text holding a backtick run) and `loose-settings` (non-object JSON text).
  - Rows now assert both, so the comment should name the fence-widening and non-object entries. It should also say the test pins both, as it does for the other branches.

- **LOW, `cmd/lym/main.go:161`, the reworded registration comment:** the line is too long.
  - It is about 152 characters, over the prose skill's 120-character limit for one line.
  - The latest repo commit, "break wrapped and over-long comments", shows over-long comments are being fixed in this change set.
  - Break it at the clause: one line for "merges heddle.json by entry through heddle's Bolt replayer," and one for "which also claims HEDDLE.md and renders it afresh after each replay."
  - Neighbouring comments in the same block are also long, but they were not touched by this fix.

Checked and consistent:
- `doc.go:33` on `RenderOverview` has no claim the new warning note contradicts.
- `render.go:96` still reports `DerivedWarning`, so the overview note matches `Render`'s behaviour.
- The `PATTERN` bullet text agrees with the replayer's claim on `HEDDLE.md`.

Tool calls:
1. Grep of `doc.go` and `render.go` for `HEDDLE.md`, `DerivedWarning` and "left out", plus a line-length check of `cmd/lym/main.go`. My context lacked `doc.go` and the real length of the edited comment.
