# Fixer report — plan, round 2

Every finding's premise held against the cards and `contracts/stencils/claims_test.go:691`, and each was fixed.

## Changes

- **F1** (`10-pr-align-producer.md`, step 7): a `Snapshot()` error is handled as a `Begin` error is, with nothing spawned, a note naming the error, and `Done`.
  `TestPRAlign` gains the row "A `Snapshot` closure that errors spawns nothing, writes the note naming the error and ends `Done`".
- **F2** (`07-parent-review-gate-alignment.md`, step 3): an `Aligner.Poll` error is treated as a `Done` poll with no verdict and the error text as its reason, so it falls back to delivery.
  `closeAlignment` logs an `Aligner.Close` error on every path and still settles the record through `SettleAlignment`.
  `TestGate_Alignment` gains the Poll-error and Close-error-on-fallback rows.
  In `10-pr-align-producer.md`, step 8: a `Poll` error closes the attempt, writes a note naming the error and is `Done`, and a `Close` error on any path is logged while the note is still written.
  `TestPRAlign` gains the Poll-error row.
- **F3** (`10-pr-align-producer.md`, step 9): every `AlignmentNote` records the re-read pull request number and head from apply time.
  A note written with no re-read records the step-4 values.
  This covers the `Snapshot`, `Begin` and `Poll` error notes and the pause note; the `Poll` error note is added to the fix's list because F2 introduces it.
  The moved-head row now also asserts that the note carries the new head.
- **F4**: `06-alignment-session.md`'s `TestSession` gains rows (a) to (d).
  These cover a failed strand record through a `Start` hook that replaces the attempt directory with a file, a shuttle timeout result through the watch, a `Wait` error giving `CauseDied`, and a second `Close` that changes nothing.
  `10-pr-align-producer.md`'s `TestPRAlign` gains rows (e) to (h).
  These cover a 404 or 422 answer ending `Done`, a matching rejection, the reuse of an open attempt through `Begin` with no new spawn, and a `Config` error returned.
- **F5** (`10-pr-align-producer.md`, step 9's reject branch): `MarkApplied(reject)` now runs before `RecordRejection`, and the card states why.
  The first-reject row asserts the order.
  **Fixed differently** in one detail: the fix asked for the order to be read from "the fake session's recorded order".
  The fake session never sees the rejection file, which `RecordRejection` writes outside it, so it cannot order the two by recording calls.
  The row instead has the fake record that no rejection file existed when `MarkApplied(reject)` ran.
- **F6** (`12-orch-stencil-alignment.md`): under the `wordingClaims` bullets, the claim forbidding `alignment-agent` keeps its `mustNot`, and its why is rewritten to say that the stencil names the alignment agent in prose and never by its board-note slug.

## Disputed

None.

## Deferred

None.
