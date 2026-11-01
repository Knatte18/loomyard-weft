# Fixer report — plan round 2

## Changes

- **F1** (fixed as proposed): card 7 lists `contracts/specs/loom-plan-spec.md` under **Edit:**, after `internal/loomshed#NewLensReviewPlanGate`; `internal/loomshed/doc.go` is the only **Extends:** bullet.
- **F2** (fixed as proposed): card 5 gains an **Extends:** group holding `contracts/specs/loom-plan-spec.md`, and an Intent bullet saying the spec's part-scope note, the one card 4 adds, also states that under a part scope `glyph-not-found` leaves out a target path the part's external exports hold and `caller-uncovered` is left to the joined plan's `plan` gate.
- **F3** (fixed as proposed): card 9's **Edit:** group gains `internal/seatengine#stepEnder`, `#stepEnder.end` and `#stepEnder.fired` after `#seatSpec`, and its Intent says their comments name the seat that ends the step, a required advisor or a writer.
- **F4** (fixed as proposed): card 18 says the help paragraph "Every mode but --require-approved also runs …" states that `--part` runs those context checks without the decision record and leaves `pattern-changes-mismatch` and `pattern-line-unprescribed` to the joined plan's `plan` gate, and that "five modes" and "The four flags are mutually exclusive" count `--part`, as six modes and five flags.
- **F6** (fixed as proposed): card 14's `TestPartsPlanTable` clause drops "the text of the four plan-format blocks", keeping the part name, the plan directory and no `{{`.
- **F7** (fixed differently): card 9's `TestEngine_Run_RequiredAdvisorEndsTheStep` sentence now has two writer rows, a failed gate (`seatRun.watch`) and a failed start (`Engine.Run`), each asserting `EndedStep`, `Writer` and `Outputs` and nothing queued.
  The proposal said the other five ends are covered by the advisor rows through the same predicate.
  That holds for only four of them: the not-found-at-resume end goes through `Engine.Resume`, not `seatRun.watch`.
  The card therefore says the died, timeout, wait-error and Accept-error ends reach `Seat.endsStep` at `seatRun.watch`, and that the resume end is card 10's, whose line 37 adds its writer variants.
- **F8** (fixed as proposed): card 18 drops the `TestValidatePartsCmd` rows "an absent parts file failing with `parts-missing`" and "the same findings as `loomshed.NewPartsGate` reports", and drops `TestValidatePlanCmd_Part`'s absent-parts-file row.
  The self-check's change to line 40, listed below, makes the unknown-part row cover the envelope of every `ValidatePart` error.
- **F9** (fixed as proposed): card 12's Step 3 recipe keeps the blank line after the heading and the blank line after the three opening lines.
- Parent instruction (creel #218, the added decision "The part check leaves pattern-line-unprescribed to the joined plan"): card 7's Intent bullet that clears pc's `DecisionRecordPath` names that entry as the reason for clearing it.

`lym loom validate-plan --overlay` with `{}` reports `ok: true` with informational findings only, after the review fixes and again after the self-check fixes.

## Self-check

- fix-claims: no findings.
- fix-consistency, LOW: card 7's "the parent's added decision on the part check" now fits two entries. **Fixed as proposed**: line 51 says "the parent's added decision that the part check flags a missing export".
- fix-proposal, LOW: after F8, card 18's line 40 names only `ErrPartUnknown` as a refused error, so the absent-parts-file refusal at the CLI was untested. **Fixed with own change** (the lens's second alternative): line 40 says every error `ValidatePart` returns is refused with its text in the same error envelope, an `ErrPartUnknown` one naming the held parts and an absent parts file's naming its way forward. The unknown-part row now pins that envelope, so the dropped row stays dropped.
- fix-affected-set, LOW: card 9 left the advisor-only comments of `SeatResult`'s `EndedStep`, `newSeatRun` and `seatSpec` unreworded. **Fixed as proposed**: card 9's Intent says those three comments are reworded for a writer.

## Disputed

None.

## Deferred

None.
