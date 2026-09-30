# Card 5 — rebaseline-engine

**Edit:**
- `internal/websterengine/beginbatch.go`
- `internal/websterengine/runlevel.go`

**Create:**
- `internal/websterengine/rebaseline.go`
- `internal/websterengine/rebaseline_test.go`

**Uses:**
- `internal/websterengine/state.go`

**Intent:**
Implement D10's engine half: accept the on-disk plan as the run's plan without discarding any batch record.

`rebaseline.go` declares:

- `ErrRebaselineCardSetChanged`, a sentinel.
- `RebaselineDeps` (`Plan *planparser.Plan`, `Batches []batcher.Batch` sequenced from that plan, `State *State`) and `RebaselineResult` (`PreviousFingerprint`, `Fingerprint`, `BatchesKept int`).
- `Rebaseline(deps RebaselineDeps) (*RebaselineResult, error)`.
  For every record in `State.Batches` except the integration key, it compares the record's card set (`Cards`, or `[NN-<Slug>]` for a record without `Cards`) with the card set the edited plan's batch of that number now holds.
  A batch number the edited plan no longer has, or one whose card set differs, refuses with an error wrapping `ErrRebaselineCardSetChanged` that names every such batch with its recorded and current card set, and ends `way forward: restore those cards in the plan, or reset the branch to the run's start commit <earliest recorded StartSHA> with git and run "lyx webster run --fresh"`.
  Otherwise it restamps `State.PlanFingerprint` through `restampFingerprint` and leaves every other field untouched.
  It never saves; the caller holds the state-mutation lease and saves, as for the bracket verbs.

Fingerprint messages (D10):
`ErrFingerprintMismatch`'s wrapping messages in `BeginBatch` and in `Run`'s entry refusal name `lyx webster rebaseline` first — "if the edit keeps every begun batch's cards, run `lyx webster rebaseline` to accept it" — and `--fresh` only with its precondition: "otherwise reset the branch to the run's start commit and run `lyx webster run --fresh`".
Update the two existing assertions on these messages in `beginbatch_test.go` and `runlevel_test.go` if they pin the old text.

`rebaseline_test.go` (`//go:build integration`, reusing the begin fixture) covers the must-cover scenarios:

- A foreign plan edit mid-run (a later, unbegun card's Intent changed after batch 1 was begun and recorded done): `BeginBatch(2)` refuses with `ErrFingerprintMismatch` and a message containing `lyx webster rebaseline`; `Rebaseline` succeeds; `BeginBatch(2)` then proceeds, and batch 1's record (status, digest, StartSHA) is unchanged.
- An edit that removes a recorded card, and one that renumbers a begun card (a different slug under a begun number): `Rebaseline` refuses with `ErrRebaselineCardSetChanged`, the message names the batch and both card sets and contains `--fresh`, and `State.PlanFingerprint` is unchanged.
- A regrouped begun batch (hand-built `Batches` putting two cards in the begun batch's number): refused the same way.
- A legacy record without `Cards` whose single card still matches: accepted.

**ImpactSummary:** Two fingerprint-refusal texts in begin-batch and run entry change; the ErrFingerprintMismatch sentinel and its errors.Is matches are unchanged.
