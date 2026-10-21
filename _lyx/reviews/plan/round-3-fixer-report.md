# Plan review round 3 — fixer report

Every finding's premise was checked against the code at HEAD and held.
`lyx loom validate-plan` reports `ok` after the fixes.

## Changes

- **F1** (BLOCKING), fixed as proposed.
  Card 13 lists `internal/testkit/envkit#nilLegal` in Edit.
  Its Intent says `nilLegal` gains `DocumentSubject`, `CodeSubject`, `MergeSubject`, `MergeInOnly`, `WeftHead` and `RejectReview`, so batten's wiring, and loom's wiring before card 15, keep passing the `NilSeams` audits.
  Card 14 lists `internal/testkit/envkit#nilLegal` and `internal/battencli#intentionallyNil` in Edit.
  Its Intent says `nilLegal` gains `Darn.RecordRejection` (nil records nothing), and batten's `intentionallyNil` gains `Rework.KeepReviewRecords` with the reason `loom-only seam, batten has no rework segment`.
- **F2** (LOW), fixed as proposed.
  Card 2 lists `internal/shedengine#Goto` in Edit.
  Its D1 paragraph says `Goto`'s doc comment stops saying its `OutcomeGoto` entry ends the target segment's bounce episode.
- **F3** (LOW), fixed as proposed.
  Card 16 lists `internal/loomcli/doc.go` in Edit.
  Its Intent says the `# Arbiter` paragraph's list of unsound-resume preconditions gains a goto target the told guard refuses.
  It also says the tackler and arbiter re-launch paragraphs state that a guard refusal refuses the attempt and the driver hands back with the refusal text.
- **F4** (LOW), fixed differently.
  The proposal named all three status help texts, including `shedVerbTexts`.
  I did not change `shedVerbTexts`, because its status text (`internal/shedcli/cli.go:76`) defers to the recipe's own status verb and lists no envelope key.
  Card 5's Intent now says the status help texts of `loomVerbTexts` (`internal/loomcli/cli.go:381`) and `battenVerbTexts` (`internal/battencli/cli.go:169`) name `pause_reason` among the envelope's keys, and that shed's text stays as it is.
- **F5** (LOW), fixed as proposed.
  Card 18 lists `internal/loomcli#dispositionForHandshake` in Edit.
  Its Intent now names the `awaitRunLockHalted` paragraph of that function's doc as the text being rewritten.

## Self-check

- fix-consistency, LOW: card 13 gave `RejectReview` the no-subject nil reason, while the card's own seam list documents it as "optional, nil for none".
  Fixed as proposed: the sentence now gives the first five seams the no-subject default and `RejectReview` the no-review-file default.
- fix-claims: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
