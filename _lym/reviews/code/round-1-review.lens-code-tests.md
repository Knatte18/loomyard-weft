```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test that pins it.

**Behaviour → pinning test**

| Behaviour | Test | Verdict |
|---|---|---|
| A plain promote of a Sketch (sketch label, record stored) takes raddle, keeps the record and flag, and is released (status nil, not draft) | `TestDiscussedPromotion` row `PlainPromoteOfASketchTakesRaddle` (`sketch_test.go:258`); the status check is at line 329 | pinned |
| The same by handle (`S1`) | row `PlainPromoteOfASketchByHandleTakesRaddle` (line 262) | pinned |
| A plain promote of a discussed note whose sketch label was stripped takes raddle and gets the label back | row `PlainPromoteOfAStrippedNoteTakesRaddle` (line 266) | pinned |
| A promote with `--recipe raddle` stays accepted | rows `RaddlePromoteOfASketch` and `RaddlePromoteOfAStrippedNote` (lines 250 and 254, unchanged); `slugrule_test.go:219` and `:248` | pinned |
| Any other recipe on a Sketch is refused, naming the plain promote | rows `DarnPromoteOfASketchIsRefusedByTheLabelRule` and `DarnPromoteOfAStrippedNoteIsRefused`, which assert `wayForward` | pinned |
| A discussed note whose record file is missing, with no sketch label, gets no recipe on a plain promote | row `PlainPromoteOfANoteWithAMissingRecordFileTakesNoRecipe` (line 293) | pinned |
| A note that is not discussed keeps today's recipe handling on a plain promote | no new row; `store_test.go:1869` (`TestPromote`) and `board_test.go:431` cover plain promotes through `Store.Promote` and `Board.Promote` | pinned (existing) |
| `SketchPromoteCommand` text in the promotion-rule refusal | `wayForward` assertions in `sketch_test.go` | pinned |
| `SketchPromoteCommand` text in the label-rule refusal | `labels_test.go:31` | pinned |
| `SketchPromoteCommand` text in batten's start guard | `startguard_test.go:458` | pinned |
| The refusal text is correct when reached from upsert | none | see finding 1 |
| A plain promote of a Sketch through the CLI (`lym board promote S3`, no `--recipe`) | none; the engine rows cover the logic, and the bare-handle lookup in `resolveLookup` is unchanged | not a gap, since the behaviour is the engine's |
| Help, flag and doc text | static prose | exempt |

**Findings**

1. **MEDIUM, class `design`** (coverage gap). `internal/boardengine/sketch_test.go:289-296` and `store.go:783-790`.
   - **The gap:** no row covers a plain promote of a note that still carries the sketch label while its record file is missing.
   - **Why it matters:** `recordHeld` is false for that note, so the recipe stays empty, and `checkSketchLabel` (`labels.go:66-67`) refuses it with a way forward that is the same plain promote.
   - **What the rows cover:** both missing-record rows use `recordMissing`, which starts from `stripped` and so has no label.
   - **Before the diff:** `--recipe raddle` worked for this note. No test pins either the refusal or a fix.
   - **Fix:** add a row, with a labelled note and a missing record, that pins whichever behaviour is chosen.

2. **LOW, class `design`**. `labels_test.go:28-33`.
   - **The gap:** the refusal reached from upsert is asserted only on the `prior.Discussed` branch of `checkSketchLabel`, which is the row's `prior`.
   - **Why it matters:** the new clause "which gives it recipe raddle" has no assertion for the missing-record case in finding 1, where it is false.
   - **Fix:** fold this into finding 1.

**Tool calls (1)**
- grep of `board_test.go` and `store_test.go` for plain-promote tests and the `wantDraft` and `wantRecipe` assertions. Plain promotes of undiscussed notes were not in my context, and I needed to confirm that the `sketch_test.go` table checks the released status.
