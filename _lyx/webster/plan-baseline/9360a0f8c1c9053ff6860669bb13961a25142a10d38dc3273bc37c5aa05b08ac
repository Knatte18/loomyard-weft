# Card 9 — delivered-after-verdict

**Edit:**
- `internal/parentreview#Store.RecordDelivered`
- `internal/parentreview#TestRecordDelivered`
- `internal/loomcli#reviewDeliveredVerb`
- `internal/loomcli#loomCLI.reviewCmd`
- `internal/loomcli#TestReviewVerbs_Success`

**Intent:**
D8: `lyx loom review delivered`, with or without `--failed`, on a round whose verdict is recorded succeeds without writing `delivery.json` or logging the failure Warn, and says the round is settled.
After an approve the discussion commit may already have run, so a late stamp left `_lyx` dirty until the next status commit.

`Store.RecordDelivered` keeps its refusal order: no open request refuses `ErrNoOpenRequest`, and an expired request `ErrExpired`.
It then writes the delivery under the round lock, reading `verdict.json` with `readHeldJSON` inside the update the way `MarkExpired` does; a recorded verdict returns `ErrVerdictRecorded` and writes nothing.
The `notice not delivered` Warn is logged only after a successful write, so a settled round logs none.
The doc says so.
It skips only the stamp of a round that can no longer use it: an open round with no verdict is stamped as before, and an expired round still refuses.

`reviewDeliveredVerb` maps `ErrVerdictRecorded` to a success envelope, exit 0, carrying `slug`, `action: delivered`, `settled: true` and a `message` saying the round already carries a verdict, so nothing is recorded and nothing more is needed;
every other error still goes through `reviewStoreErr`.
The `delivered` command's Long help in `loomCLI.reviewCmd` adds that on a round with a verdict it records nothing and reports the round settled.

Tests:
`TestRecordDelivered` records an approve on an open round, then asserts `RecordDelivered("")` and `RecordDelivered("no such agent")` each return `ErrVerdictRecorded`, `delivery.json` does not exist, and the captured log holds no `notice not delivered` line, using the `logger.SetOutput` capture `gate_test.go` uses.
`TestReviewVerbs_Success` gains "delivered after a verdict" and "delivered failed after a verdict" subtests: exit 0, `settled` true in the envelope, and no `delivery.json` in the round.
The expired refusal stays covered by `TestReviewVerbs_Expired`'s `delivered` row.

**ImpactSummary:** the delivered verb and its store write turn a settled round into a no-op success; open and expired rounds behave as before.
