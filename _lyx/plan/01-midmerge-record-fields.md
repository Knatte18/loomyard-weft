# Card 1 — midmerge-record-fields

**Edit:**
- `internal/fabricengine#MidMergeState`
- `internal/fabricengine#MidMerge`
- `internal/fabricengine/midmerge_integration_test.go`

**Prosa:**
- `internal/fabricengine/doc.go`

**Intent:** Widen the mid-merge probe's answer so a caller can tell a parked merge's identity, not only its kind.
`MidMergeState` gains four string fields, `Verb`, `Source`, `SourceSHA` and `StartSHA`, filled only when `Kind` is `MidMergeParked`: the parked record's verb (`merge-in` or `merge`), its caller-supplied source branch, the resolved SHA it merged and the pre-merge HEAD SHA it recorded.
`MidMerge` loads the record through the existing `loadMergeState` instead of `mergeRecordExists`, so the one read answers both existence and identity; every other path, the foreign-state classification, the conflict-path unification and the error returns, is unchanged.
The field names are vocabulary-neutral, since the struct is read outside the fabric vocabulary owner set.
The two parked tests in the integration file assert the new fields: `Verb` is `merge-in`, `Source` is the branch `MergeIn` was given, and the two SHAs are the pre-merge HEAD and the source branch's SHA; the clean-pair and foreign-state tests assert the fields stay empty.
The package doc's `MidMerge` sentence says the answer now carries the parked record's verb, source and SHAs.

**ImpactSummary:** Adds fields to a read-only probe result; its callers in `internal/loomcli` ignore the new fields until card 6, and no other package reads the struct.
