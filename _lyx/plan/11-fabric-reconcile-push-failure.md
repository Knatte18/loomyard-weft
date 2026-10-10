# Card 11 — fabric-reconcile-push-failure

**Edit:**
- `internal/fabriccli#Command`
- `internal/fabriccli#runReconcile`
- `internal/fabriccli/cli_test.go`
- `internal/fabriccli/hubmutation_integration_test.go`

**Uses:**
- `plan:internal/fabricengine#Bolt.PullThenCommitWritten`
- `plan:internal/fabricengine#BoltWriteResult`
- `internal/fabriccli#errWithRecordFields`

**Intent:** D4 and D7 for `lyx fabric reconcile`.
`runReconcile` heals the hub-wide config through `Bolt.PullThenCommitWritten`, which records the file writes and the commit itself, so the handler's own `KindFileWritten` and `KindCommitCreated` appends go; a skipped heal, fetch failure or divergence, writes nothing and is reported under `hub_config_detail` with the skip's detail, which is not a failure of the verb.
A commit or push failure of the hub-wide config, or of the warp-binding record on either the `recorded` or the `present` path, no longer leaves the exit code alone: the envelope is built as today, `hub_config_detail`, `warp_binding` and `warp_binding_detail` included, and returned through `errWithRecordFields` with an error naming the first failure and the way forward, a re-run once the remote is reachable, or `git pull --rebase` in the board dir then `lyx board sync` for a rejected push; the pair-failure error keeps precedence when both occur.
`contracts/specs/refusal-spec.md` has no fabric section, so no row is added (`PATTERN-refusal-way-forward` binds webster, shed and loom).
`Command`'s `reconcile` long help drops the sentence saying a commit or push failure leaves the exit code unchanged and says instead that it fails the verb with the envelope kept, and `runReconcile`'s doc comment says the same.
`TestRunCLI_ReconcileBackfillFailureIsNonFatal` becomes `TestRunCLI_ReconcileBackfillFailureFailsTheVerb`: with the board remote unreachable the heal is skipped and reported under `hub_config_detail`, the warp-binding commit lands, its push fails, the exit code is non-zero and the envelope still carries `pairs`, `warp_binding` equal to `record_failed` and a non-empty `warp_binding_detail`; `TestRunCLI_ReconcileBacksFillsWarpBinding` is unchanged.
`TestRunCLI_HubMutationScenario`'s step that removes the committed hub-wide configs and points the remote at an unreachable path now expects the heal skipped with `hub_config_detail` naming the fetch, the config loader's own failure making the exit non-zero, and the files not healed; the step's comment is rewritten with it.

**ImpactSummary:** One CLI verb's exit code flips to non-zero on a hub-config or warp-binding push failure, with two tests pinning the old code listed; the envelope keys are unchanged.
