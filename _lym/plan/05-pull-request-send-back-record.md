# Card 5 — pull-request-send-back-record

**Edit:**
- `internal/pullrequest#Record`
- `internal/pullrequest#TestRecord_RoundTripAndUpdate`

**Uses:**
- `//internal/pullrequest/doc.go`

**Intent:** Give the run's pull-request record the four fields D4's verb and row share.

- `SendBack string` (`send_back`, omitempty): the send-back mark, the absolute path of the findings file the verb stored.
- `FixHereCount int` (`fix_here_count`, omitempty): the findings PR-Review fixed in place since the last send-back.
- `SendBacks int` (`send_backs`, omitempty): the run's send-backs so far, which numbers the next `pr-findings-<n>.md`; no clear resets it.
- `CountedSets []string` (`counted_sets`, omitempty): the sha256 of each findings file `FixHereCount` already counts, so a repeated fix-here call over the same file counts it once; it is cleared with `FixHereCount`.

The record reads and rewrites through `internal/state`, so a record from an older run, without these keys, reads as zero values (`PATTERN-tolerant-records`).
`TestRecord_RoundTripAndUpdate`, tier0, round-trips the four fields and reads a file without them as zero.

**ImpactSummary:** Four omitempty fields on the pull-request record; existing readers and writers are unaffected.

**Commit:** `5: pull-request-send-back-record`
