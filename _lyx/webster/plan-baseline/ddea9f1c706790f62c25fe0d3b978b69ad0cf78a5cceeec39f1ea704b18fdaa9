# Card 6 — FABRIC_SKIP_* env vars and the records-sibling refusal

**Edit:**
- `internal/fabricengine/fabric.go`
- `internal/fabricengine/spawn.go`
- `internal/fabricengine/commit.go`
- `internal/fabricengine/doc.go`
- `internal/websterengine/doc.go`
- `internal/webstercli/reset.go`
- `internal/configcli/configcli_integration_test.go`
- `internal/fabriccli/weftskippush_integration_test.go`
- `internal/fabricengine/checkout_rollback_test.go`
- `internal/fabricengine/fabric_test.go`
- `internal/fabricengine/hubcontainment_integration_test.go`
- `internal/fabricengine/reconcile_stale_registration_test.go`
- `internal/fabricengine/remove_junctions_integration_test.go`
- `internal/fabricengine/spawn_test.go`
- `internal/fabricengine/unwire_test.go`
- `internal/webstercli/acceptaudit_test.go`
- `internal/webstercli/cli_test.go`
- `internal/webstercli/reset_test.go`
- `internal/webstercli/sync_integration_test.go`
- `internal/webstercli/verbs_test.go`
- `internal/battencli/lifecycle_integration_test.go`
- `internal/fabriccli/cleanhub_integration_test.go`

**Prosa:**
- `tools/sandbox/SANDBOX-FABRIC-SUITE.md`

**Intent:**
D4, env and wording part.

`WEFT_SKIP_GIT` and `WEFT_SKIP_PUSH` become `FABRIC_SKIP_GIT` and `FABRIC_SKIP_PUSH`, as a hard cut with no fallback read of the old names: `EnvSyncOptions` and `SpawnDetachedPush`'s own gate read the new names, every test that sets them sets the new names, and every comment, long help and package doc naming them (`commit.go`, `spawn.go`, `fabricengine/doc.go`, `websterengine/doc.go`, `webstercli/reset.go`'s help) names the new ones.
A final grep over the module, scripts and docs included, finds no `WEFT_SKIP_`.
What the hard cut lets through, and its bound: a process that still sets an old name gets real git and push work; the only setters are this repository's tests and sandbox, changed in this commit, and the variables have no production consumer.

`RequireWarpWorktree`'s refusal says "records sibling" where it said "weft sibling".
The tests that assert that wording, `battencli`'s `lifecycle_integration_test.go` ("weft sibling") and `fabriccli`'s `cleanhub_integration_test.go` ("weft sibling of a pair"), assert the new text; `fabric_test.go` asserts only the wrapped `ErrNotAWarpWorktree`, so it is a target for `FABRIC_SKIP_*` alone.
`tools/sandbox/SANDBOX-FABRIC-SUITE.md`'s instruction to confirm a refusal "naming the weft sibling" names the records sibling instead; a grep for the old phrase finds no other assertion or instruction.

The renamed variables are covered by the existing skip-gated tests, which now set `FABRIC_SKIP_*` and would push for real if the gate still read the old names.

**ImpactSummary:** two env-var names and one refusal message change; their readers are fabricengine's two gates, and their setters and asserters are tests changed here.
