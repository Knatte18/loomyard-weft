# Fixer report — plan, round 1

Card numbers below are the post-fix numbers; F7 inserted a card at 4, so the review's cards 4–11 are now 5–12.

## Changes

- **F1** (fixed as proposed): Card 7 (was 6), Intent step 1, now says an `Aligner.Spawn` error is logged, recorded through `RecordAlignment` as `fallback` with the error as `Reason`, rewrites the brief and continues into today's delivery in the same evaluation, never returned.
  `TestGate_Alignment` gains the failing-`Spawn` row.
  Card 10 (was 9), step 7, now says a `Begin` error writes a note with the error as reason and ends `Done`, and `TestPRAlign` gains the failing-`Begin` row (no record, the note, `Done`).
- **F2** (fixed as proposed): card 10's PR-Gate bullet now starts the reason with the clause, report path first: "alignment report <path> did not settle this pull request - <reason>; " followed by the existing reason, and names the `noticeErrorMax` cut it survives.
  The `TestPRGate_Verdicts` note row asserts the reason starts with the clause.
- **F3** (fixed as proposed): card 6 (was 5) `TestSession` gains a failing-`Start` row and a failing-`StopStrand` row.
- **F4** (fixed as proposed): card 10's `TestPRAlign` gains the 5xx, no-pull-request, closed-pull-request and differing-`TaskHead` rows.
- **F5** (fixed as proposed): card 5's (was 4) stencil list now says every escalate fills `question`, naming the peer and overlap or the question and its open record; `TestReadVerdict`'s acceptance rows include an overlap escalate with a question.
- **F6** (fixed as proposed): card 10's `NotePath` doc reworded as the fix gives; card 8's (was 7) `alignmentSettings` line now credits the PR-side wiring card, which after the renumber is card 10.
- **F7** (fixed as proposed): `lyx loom insert-card 4 shedrun-alignment-paths` added `04-shedrun-alignment-paths.md`, holding `AlignmentDir`, `AlignmentRel`, the two `shedrun` test edits, `doc.go` and the shedrun Intent paragraph, all removed from card 5.
  Every prose mention insert-card reported was renumbered by hand; cards 8 and 10 list the new symbols under `Uses:`, and the overview's summary sentence names the record paths.
  `lens-review.md` keeps its old numbers, since it records an earlier round.
  `lyx loom validate-plan` reports `ok: true`.

## Disputed

None.

## Deferred

None.
