# engine-usage-readers: tokencount reads through the shuttle engines

All five batches reported `status: done`.
Batch 03 also carried card 04, which the batcher grouped with card 03.
No warnings came back on any `record-batch`.

## Batches

- **01-neutral-transcript-types** (head `f387c3089`): `shuttleengine` declares the neutral transcript, usage, pricing and read-set types in `transcriptread.go`.
  Deviation list: `internal/shuttleengine/transcriptread.go`.
- **02-claude-transcript-reader** (head `18f5d69b8`): `claudeengine` locates and reads Claude transcripts through one parse shared with `SessionUsage`.
  The tokencount-run fixtures and the `shuttlefake` golden were added.
  Deviation list: `claudeengine/transcriptread.go`, `transcriptread_test.go`, `usage.go`, `usage_test.go`, `internal/testkit/shuttlefake/shuttlefake.go`.
- **03-claude-pricing**, with card 04 (head `86a739e6c`): `claudeengine` prices a message from its model and neutral usage.
  `shuttleengines` hands out each engine's read capabilities through `RecordReaders`, behind adapters.
  Deviation list: none.
- **05-tokencount-reads-through-engines** (head `a9df3858d`): tokencount counts sessions, forks and orch through the read set and drops its Claude parser, price table and `-projects` flag.
  Deviation list: `tools/tokencount/cost.go`, `count.go`, `count_test.go`, `main.go`.
- **06-calibration-engine-instruction-files** (head `2b6e5d5be`): calibration reads the Webster session engine's instruction files instead of `CLAUDE.md`.
  Deviation list: `tools/tokencount/calibrate_test.go`, `count_test.go`, `main.go`.

## Scope guard

The scope guard listed the files above as changed outside the batches' declared targets.
It is informational only, and no batch was failed on it.
