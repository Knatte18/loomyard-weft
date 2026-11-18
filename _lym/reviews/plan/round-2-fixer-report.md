---
dispositions:
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
---

# Round 2 fixer-report

## Changes

- **F9, fixed as proposed.**
  - Card 3 (`_lym/plan/03-drift-verify-loosened-checks.md`): `internal/battencli#intentionallyNil` is listed under **Edit:**.
    The Intent's seam list says the table gains `Drift.ChangedFiles` and `Drift.CommitMessages` with the reason `loom- and darn-only seam, batten has no Drift row`.
    It also says `Drift.ReviewBase` needs no entry, because `NilSeams` skips the paths in `envkit`'s `nilLegal`.
  - Card 9 (`_lym/plan/09-pr-review-send-back-row.md`): `internal/battencli#intentionallyNil` is listed under **Edit:**.
    The Intent says the table gains `Landing.PullRequest.ShedStatus` with the reason `loom-only seam, batten lands no branch`.
  - `TestWire_EverySeamFilled` stays under **Uses:** in both cards.
- **F10, fixed as proposed (the first of the review's two options).**
  - Card 7 (`_lym/plan/07-bouncer-review-inputs.md`): `internal/shedrecipe#captureReview` is removed from **Edit:**.
  - The `TestBurlerReviewEntry_RubricStencil` row now builds the Code-Review row with `burlerReviewEntry` itself and writes the full generation record naming `pr-findings-1.md` after the build.
    It then calls the producer and reads the rubric from the `BurlerHalfRunner` fake's `GotReviewProfiles`, leaving `captureReview` untouched.
- **Self-check, fork-affected-set finding, fixed as proposed.**
  - Card 9: `internal/loomcli#intentionallyNil` is listed under **Edit:**.
  - The Intent says the table gains `Landing.PullRequest.ShedStatus` with the reason its other `Landing.PullRequest.*` entries give (`internal/loomcli/wiring_seams_test.go:37-57`).
    `loomCLI.wire` leaves `Env.Landing` to run.go, so `loomcli#TestWire_EverySeamFilled` would otherwise fail at card 9's gate.

`lym loom validate-plan` passes with only the existing informational finding on card 11.
The reviewer's `--overlay` check was not run as such: the edits were already on disk, so the run read the edited cards directly.

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-proposal: no findings.
- fix-affected-set, BLOCKING: card 9 misses `loomcli`'s own `intentionallyNil` table. Fixed as proposed (see Changes).

## Disputed

None.

## Deferred

None.

## Rulings

None.
