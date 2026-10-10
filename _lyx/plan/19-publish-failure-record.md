# Card 19 — publish-failure-record

**Create:**
- `internal/verifytree#FailureKind`
- `internal/verifytree#FailureKindPlanVerify`
- `internal/verifytree#FailureKindPublishVerify`
- `internal/verifytree#FailedTest`
- `internal/verifytree#PublishFailure`
- `internal/verifytree#WritePublishFailure`
- `internal/verifytree#ReadPublishFailure`
- `internal/verifytree#RemovePublishFailure`
- `internal/verifytree/failure_test.go`

**Edit:**
- `internal/verifytree#Paths`
- `internal/verifytree#NewPaths`
- `internal/verifytree/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Declare the Publish failure record (D11), the one never-tracked file Publish leaves when its plan verify or `publish_verify` fails and a later passing Publish removes.
`Paths` gains `PublishFailure` and `PublishFailureLog`, which `NewPaths` places in the verify directory beside the log (`PATTERN-durable-vs-ephemeral-state`).
`PublishFailure` holds the failing verify as a `FailureKind` from the closed set above, never a command string; the failing tests as `FailedTest` entries (import path and test path); the path of the log copy; HEAD; and the merge-in commit Publish made, empty when it made none.
`WritePublishFailure` copies the current `p.Log` to `p.PublishFailureLog`, so a later verify in the same directory cannot overwrite the evidence, sets the record's log path to that copy, and writes the record as YAML.
`ReadPublishFailure` reports false for an absent record and an error for a malformed one; `RemovePublishFailure` removes both files and is a no-op when they are absent.
`failure_test.go` (tier 1) pins the round trip, the log copy surviving a rewrite of `p.Log`, the absent read and the idempotent removal; the verifytree budget row rises for it.
`doc.go` names the record and who writes, reads and removes it.

**ImpactSummary:** Two new path fields and a codec in verifytree; no caller writes or reads the record until the next cards.
