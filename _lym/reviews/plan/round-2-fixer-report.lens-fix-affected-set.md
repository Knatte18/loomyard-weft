```yaml
lens: fix-affected-set
outcome: reported
```

Task: for each set-shaped finding (F1, F2, F3, F4, F8), rerun its search and check that the fix covers every member.

Unchecked: none. I reran the searches for F2, F3 and F4. F1's set is one spec sentence plus three rows, and F8's is the test rows. Card 7 and card 18 already show both sets in full, so neither needed a search.

**[LOW] The F3 fix leaves three advisor-only comments stale in symbols card 9 already edits.**
- **Changed text:** card 9 (`_lym/plan/09-seat-writer-kind.md`), the added sentence "The `stepEnder` type and its `end` and `fired` methods are reworded the same way: their comments name the seat that ends the step, a required advisor or a writer."
- **Summary:** F3's set is every seatengine comment that still says only a required advisor ends the step, or that only an advisor takes the seat behaviour a writer now shares. Three members of that set are in symbols card 9 lists under **Edit:**, but no sentence of the card rewords their comments:
  - `internal/seatengine/engine.go:28`, the `SeatResult` field comment: "EndedStep is true for a required advisor whose end ended the step before the chair's." A writer's result now carries `EndedStep` too, and card 9's test rows assert it. The card adds fields to `SeatResult` but never rewords this one.
  - `internal/seatengine/engine.go:111-112`, `newSeatRun`: "returns the run of an advisor that started … unless it is required, which ends the step on ender instead." Card 9 edits `newSeatRun` for its outputs only.
  - `internal/seatengine/prompt.go:159`, `seatSpec`: "only an advisor keeps its pane, and an advisor holds its turn end quietly unless it is Required." Card 9 makes a writer keep its pane and hold quietly, but the card states only the code change, not this comment.
- **Evidence:** the grep output above. Card 9's "other advisor-only comments" sentence and its new `stepEnder` sentence name neither these comments nor these fields. Card 10 owns `Engine.join` (engine.go:343), so that comment is out of this set.

F2: the spec's `caller-uncovered` row 35 (`loom-plan-spec.md:513`) and the `glyph-not-found` text are both covered by card 5's new sentence on the part-scope note. No other spec row names either check in a part-scope sense.

F4: validate-plan's mode count and context-check claim appear only in `internal/loomcli/validate.go:164`, `:176-178` and `:191`. Neither `internal/loomcli/doc.go` nor `docs/overview.md` repeats them, so card 18's rewording covers the whole set.

Tool calls:
1. Grep of `internal/seatengine` for comments naming a required advisor. My context held only engine.go's `stepEnder` lines, not the whole set.
2. Grep of `loom-plan-spec.md` for `caller-uncovered`, `glyph-not-found`, `require-approved` and `pattern-line-unprescribed`, to rerun F2's search. My context did not hold the spec.
3. Grep of loomcli's `doc.go`, `validate.go` and `docs/overview.md` for mode-count and context-check claims, to rerun F4's search beyond `validate.go`.
4. Read of card 10's target lists, card 9's `SeatResult` and `newSeatRun` lines, and `prompt.go`'s `seatSpec` comment, to tell card 10's members from card 9's. My context lacked card 10 and `prompt.go`.
