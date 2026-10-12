# Card 14 — bouncer-blocking-and-merge-note

**Create:**
- `internal/shedadapters#RubricNotes`
- `internal/shedadapters#Bouncer.rubricNotes`
- `internal/shedadapters#Bouncer.openBlocking`

**Edit:**
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#Bouncer.settleUndecided`
- `internal/shedadapters#ReadRubric` -> `func ReadRubric(stencilsDir, name, specsDir string, notes RubricNotes) (string, error)`
- `internal/shedadapters#Bouncer.runSeedSpawn`
- `internal/shedadapters#Bouncer.judgeCall`
- `internal/shedadapters#TestReadRubric`
- `internal/shedadapters#TestReadRubric_EmptyStencilsDirIsAnError`
- `internal/shedadapters#TestBouncer_AutoAccept`
- `internal/shedrecipe#burlerRoundProfile` -> `func burlerRoundProfile(cfg Config, stencilsDir, specsDir string, notes shedadapters.RubricNotes) (burlerengine.Profile, error)`
- `internal/shedrecipe#burlerReviewEntry`
- `contracts/stencils#TestLoomRubrics`

**Intent:**
Two Bouncer changes the Merge-Review perch needs, kept generic and off by default.

D3, the BLOCKING escalation: `BouncerConfig` gains `BlockingEscalates bool`.
In `settleUndecided`, when the budget is spent and `AutoAccept` would settle the round as an automatic accept, a set `BlockingEscalates` first asks `openBlocking(round)`, which reads the round's ledger and reports whether an entry is `open` at severity BLOCKING.
An open BLOCKING entry escalates with cause `budget` instead, an awaiting stop at the Bouncer row the auto-accept did not settle.
A ledger read error escalates too, the stricter side.
Non-BLOCKING open findings are accepted and carried into `## Open risks` as any auto-accept does.
Bound: the flag only turns some auto-accepts into escalations; it never accepts what escalates today, and with it unset every row behaves as before.

The merge review note: the rubric fill gains a third optional marker, `merge_review`, so a rubric can render the merge record (card 10).
`ReadRubric` takes a `RubricNotes{PublishFailure, WebsterRecord, MergeReview}` in place of its two note strings, and each blank note fills its marker as `none`, as today.
`BouncerConfig` gains `MergeReview func() string`, nil-is-absent like `PublishFailure`.
`rubricNotes()` gathers the three notes for `runSeedSpawn` and `judgeCall`.
`burlerRoundProfile` takes the same struct, and `burlerReviewEntry` passes its two notes in it until card 16 adds the third.

`TestBouncer_AutoAccept` gains rows: budget 1 with `AutoAccept` and `BlockingEscalates`, where a CONTINUE whose ledger holds only non-BLOCKING open entries settles accepted and carries them over, and an open BLOCKING entry escalates with cause `budget`.
`TestReadRubric` gains rows filling and defaulting `merge_review`, and `TestLoomRubrics` passes its two notes in the struct.

**ImpactSummary:** `ReadRubric` and `burlerRoundProfile` are re-signed with every caller on this card; the new Bouncer flag and note are absent on every existing row.

**Commit:** 14: shedadapters: BLOCKING-aware auto-accept and a merge review rubric note
