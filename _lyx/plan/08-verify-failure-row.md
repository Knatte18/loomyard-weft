# Card 8 — verify-failure-row

**Edit:**
- `internal/verifytree#PublishFailure`
- `internal/verifytree#TestPublishFailureRecord`
- `internal/landingshed#verifyGate.recordFailure` -> `func (g verifyGate) recordFailure(kind verifytree.FailureKind, row string, result verifytree.Result) bool`
- `internal/landingshed#verifyGate.check`
- `internal/landingshed#verifyGate.checkPublishVerify`
- `internal/landingshed#TestPublishVerify_FailureRecord`

**Intent:**
Make the Publish failure record name the row that ran the failing verify (D1), so a later card can move the plan verify to Merge-In while batten's retry still finds the halted row.
`PublishFailure` gains `Row` (`row`, omitempty): the producer row whose verify failed; a record written before the field existed decodes with it empty.
`recordFailure` takes the row and writes it; `check` and `checkPublishVerify` pass their `producer`, so today Publish writes `Publish` for both kinds.
`TestPublishFailureRecord` round-trips the field and decodes a record without it; `TestPublishVerify_FailureRecord` asserts the written row for both kinds.

**ImpactSummary:** one optional record field and an internal gate re-sign with its two in-file callers; every reader of the record keeps working.

**Commit:** 8: verifytree: the failure record names the row that ran the verify
