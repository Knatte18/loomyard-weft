```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code change set for correctness, one input per changed branch, with caller and test validity.

**1. MEDIUM: the tackler and arbiter "is 0" messages can name the wrong file.**
- Changed text: `internal/sheddriver/tackler.go`, where the message is now `"%s is 0, which turns the tackler off"` with `shed.tackler_max_spawns`. Also `internal/shedarbiter/call.go`, `"off: %s is 0"` with `sheddriver.ArbiterMaxSpawnsName()`, which is `shed.arbiter_max_spawns`.
- The value in `Settings.MaxSpawns` is not necessarily shed.yaml's. `ResolveTackler` and `ResolveArbiter` in `internal/sheddriver/config.go` take a non-nil recipe-module override first (`maxSpawns := cfg.TacklerMaxSpawns; if o.TacklerMaxSpawns != nil {...}`).
- `batten.yaml`'s template holds both keys, and a task's `run_settings` can set them too.
- Concrete input: `batten.tackler_max_spawns: 0` with `shed.tackler_max_spawns: 4`. The message says `shed.tackler_max_spawns is 0`, which is false and sends the reader to the wrong file. The old bare text made no module claim.
- A fix is to word it as the resolved value, naming that a recipe module's key or `run_settings` overrides the shed key, or to pass the owning module into the settings.

**2. LOW: the new dobby test does not pin that `runDobby` uses `wireDobby`.**
- Changed text: `internal/beamcli/cli.go`, the `wireDobby` extraction, and `TestWireDobby_StaleStampReconcilesTheHubBase`.
- `hubDobbyDeps` has the same signature as `wireDobby`. If `runDobby` passed `hubDobbyDeps` to `runDobbyWith`, the reconcile would be gone and the test would still pass.
- Removing the `ReconcileBeforeLoad` line inside `wireDobby` does fail the test.

I found nothing wrong in these:
- `MoveKeys` with segment prefixes, `inlineStrandedAliases` on the review's anchor and alias input, and the `MoveSettings`, `MoveRunSettings` and `moveBlocks` callers.
- The argument order of the reworded format strings: memory guard, tier memory cap, `--map` refusal, skipped dotnet tests, plan-parts timeout and `TimeoutKeyName`.
- The F3, F4 (apart from item 2), F5, F9 and F13 tests, which should each fail with their defect in place. I traced this by reading and did not run them against the defect.

Tool calls:
1. One `grep` for old message fragments in `_test.go` files, and a list of tests at tier 2 and above. My context showed only tier 0 and tier 1 runs. No stale assertion on the old texts turned up.
