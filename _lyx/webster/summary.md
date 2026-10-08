# Template review models: opus[medium] and sonnet[high]

Batch 10 (`10-template-review-models`) was the only batch and finished `done` at head `5e2afb7b8315e7432ac8d5bd9e97f95a0682b465`.
The loom template now sets `review: opus[medium]` and `webster_review: sonnet[high]`.
The fork also updated the tests that pin the template: `templateConfig` in `internal/loomengine/config_test.go` and `TestWire_ReviewKeysReachTheEnv` in `internal/loomcli/wiring_test.go`.

`record-batch` reported no deviations from the declared file-ops.
It returned two informational `scope-outside-plan` warnings, one for each of those two test symbols, which it said were touched outside the batch's own target glyphs.
The plan-level verify gate runs after this turn ends.
