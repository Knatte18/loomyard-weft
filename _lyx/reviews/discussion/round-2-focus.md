---
round: 2
focus:
  - "In `_lyx/discussion/decision-record.md` D7 and its pack-fragment acceptance criterion: does a fork that meets an unresolved member glyph (a Go member glyph's unit is a package directory, and D5's whole-call resolve error leaves every member unresolved) have a stated, permitted way to locate it, consistent with the quarry and grep ban for resolved entries? Does the fragment's wording of which glyphs the pack lists agree with D4, which leaves out self glyphs and paths?"
  - "In `_lyx/discussion/decision-record.md` D2: do the entry and exit conditions for logging bind lines agree with the five reasons (`card-not-done`, `batch-not-done`, `no-delta-match`, `sibling-unmatched`, `delta-unavailable`), given that `postBatchChecks` in `internal/websterengine/recordbatch.go` returns errors for a not-done card and an unmatched handle, and tolerates `ErrQuarryUnavailable` from the delta call?"
  - "In `_lyx/discussion/decision-record.md` D2: for a plan with `language: none` that declares `plan:` handles (`BindHandles` in `internal/planglyph/handle.go` returns nothing for it), does D2 state what is logged, and does the D2 acceptance criterion cover that case, and a pass that ends in an error from `DoneChecks`, `Delta` or `BindHandles`?"
  - "In `_lyx/discussion/decision-record.md` Notes for the plan writer and Constraints: is each note either a requirement the task carries (in Scope) or absent, and does each constraint cite a source that actually lists the thing it constrains?"
---

Focus entries target the text as fixed; the reviewer still checks the whole pair against the rubric.
