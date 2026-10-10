# Card 9 — stencil-sync-message

**Edit:**
- `internal/stencilcli#Command`
- `internal/stencilcli/cli_integration_test.go`

**Uses:**
- `plan:internal/fabricengine#SyncCommitMessage`
- `plan:internal/fabricengine#BinaryLabel`
- `internal/fabricengine#CommitSeededStencils`

**Intent:** D3 for the operator's forced refresh.
`lyx stencil sync` commits each subtree through `CommitSeededStencils` as today but under `SyncCommitMessage(subtree, BinaryLabel())`, so its subject is `lyx: sync <subtree> (<label>)` and D5's recognizer never matches it; the mode and `StencilSource` arguments it passes to `ForceRefresh` were set on cards 3 and 4, and the verb keeps pulling nothing before it writes, since a forced refresh is the operator's own commit and a conflict on it fails the push as today.
`sync`'s `Short` keeps naming the `-dev` build and gains the unstamped one.
`stepListAfterFirstSync` asserts the board's last commit subject opens with `lyx: sync stencils (` and `stepSecondSyncIsIdempotent` stays as is; a new serial step asserts that under the test binary, which is unstamped, a synced copy carries no `build=` key, so the next production pre-run refreshes it (D1's forced-refresh rule, whose store half card 3 covers).

**ImpactSummary:** One verb's commit subject and help summary change; no test outside `stencilcli` asserts the old `lyx: seed` subject for a sync.
