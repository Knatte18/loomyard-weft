# Card 10 — board-prune

**Edit:**
- `internal/boardcli#`
- `internal/boardengine#`
- `internal/boardengine/boardtest#`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on the board packages' test files.

`internal/boardengine` carries a large candidate set of per-field and per-verb checks, the main fold leads.
`internal/boardcli`'s two `integration` files (`cli_test.go`, `verbs_test.go`) and `internal/boardengine`'s `sync_integration_test.go` hold the tier-move leads: a verb test that only reads or writes board JSON in a temp dir may move to an untagged file, and one that syncs through git stays.
`internal/boardengine/boardtest` holds the board benchmark and sync tests; its `bench_cli_test.go` benchmark is not a test the prune removes.

**ImpactSummary:** Test files of three board packages only; no production change.

**Commit:** 10: Prune board tests
