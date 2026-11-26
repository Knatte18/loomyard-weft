```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Behaviour → pinning test**

- `configengine.KeyName` → `TestKeyName`
- Load refuses an old key (default and worktree layers) → `TestLoad_LayerStack`, `TestLoadConfig_Refuses`
- `ValidateLocal` refuses an old key; an area mapping is admitted → `TestValidateLocal`
- `Set` refuses an old key → `TestSet`, `TestDispatchSet`
- `Set` of an area-named old key with a flow mapping is admitted, and with a scalar refused → **none** (F3)
- `Unset` removes old keys, several at once → `TestSet`, `TestDispatchSet`
- Move tables name template paths → `TestLoadConfig_Loads`, `TestLoadTapestryConfig`
- Move tables hold every D1 old key → **none** (F2)
- `MoveKeys` move, drop, comments, prefix, idempotence → `TestMoveKeys`
- Reconcile moves worktree, local, hub base and prime-seed files, and reports the lines → `TestReconcileAll*`, `TestReconcileHubWideAt_*`
- Move failure is a `*FileError` → `TestReconcile_FailureNamesItsFile` (`ReconcileAll` only)
- Prime-seed or template-base move failure names the right file → **none** (F5)
- Profiles blocks move before the follow → `TestReconcileHubWideAt_SeedOnlyFileFollowsItsTemplate`, `TestEnsure_CarriesATemplateChange...`
- `MoveSettings` moved line and prefix; `MoveRunSettings` → `TestMoveSettings`
- `MoveSettings` null and both-set drops → **none** (F4)
- Reconcile envelope `moved` → `TestConfigReconcileInGitRepo`
- Walk mails moves; a pair call mails nothing → `TestEnsure_MovesOldConfigKeysAndMailsTheMoves`
- Walk mails moves when it fails after the hub-wide commit → **none** (F1)
- `run_settings` carry, rollback, unwritten clean board, skipped write runs no carry → `TestCarryMovedRunSettings_...`, `TestEnsure_CarriesMovedRunSettingsOnTheBoard`, `...UnreachableBoardRemote...`
- Moved-key refusal in `run_settings`, profiles, upsert, Seed-Child and recipe references → `TestPrepareRunSettings`, `TestProfilesCheckLoad`, boardcli `verbs_test`, `TestSeedChild_RunSettings`, `TestSettingLookup`, `TestBuild_Errors`
- Nested budget keys and routes → `TestCheckResolved`, `TestCheckLocal`
- `NestPaths` → `TestMergeParent`
- `FlagCodeAutoAccept` on nested keys → `reinterpret_test` rows `SetOn`, `SetOff`, `AreaWithoutTheKey`
- Reconcile before load: loom pre-run, shed pre-run, beam land → `TestLoomVerbs_StaleStamp...`, `TestShedVerbs_StaleStamp...`, `TestHubLandDeps_...`
- Reconcile before load: batten arm, fabric merge-in, beam dobby, webster wire, shuttleparity → **none** (F6)
- `EnsureOrWarn` way forward → `TestEnsureOrWarn_WarnsWithTheWayForward`
- Shed loop step bounds → test-only change

**Findings**

- **F1. MEDIUM, design.** `internal/hubreconcile/walk.go` (`walkHub`, the deferred `notifyMovedKeys`).
  - No test fails if the mail goes back to success-only. The parent's added entry, round-3 defect (5), orders this behaviour.
  - The only assertion on the "moved config keys" notice is `TestEnsure_MovesOldConfigKeysAndMailsTheMoves`, a successful walk.
  - The existing failure hook, `EnsureFailingAfterHubWideWrite`, fails before the hub-wide commit, and no test using it checks move notices.
  - Evidence: diff piece `046-internal-hubreconcile.diff`; grep of `internal/hubreconcile/*_test.go` (`ensure_integration_test.go:491,614`, `reinterpret_integration_test.go:167`).

- **F2. MEDIUM, design.** `internal/loomengine/config_test.go:107` (`requireMovesIntoTemplate`).
  - The test iterates the move table under test, so an entry missing from `loomMovedKeys` or `tapestryMovedKeys` fails nothing.
  - A missing entry means a hub's tuned value is neither moved nor refused. The base layers only warn, and a local prune removes it.
  - The expected old-to-new set should come from D1 as a literal.
  - Evidence: `config_test.go:107-130`.

- **F3. LOW, design.** `internal/configengine/set.go` (`setsNewArea`).
  - Neither side of the area-named-key branch has a row. `--set discussion={...}` should be admitted and `--set discussion=x` refused.
  - Every moved-key row in `TestSet` uses `old_key`, so forcing `setsNewArea` to false, or dropping its `{` check, passes.
  - Evidence: `026-internal-configengine.diff` (`set.go` hunk and `set_test.go` rows).

- **F4. LOW, design.** `internal/configreg/runsettings.go` (`MoveSettings`, the `MoveDroppedNull` and `MoveDroppedNewSet` cases).
  - No row covers a profile block or `run_settings` object holding a null old key, or both the old key and its new path. D6 asks for D5's rule there.
  - `TestMoveSettings` only asserts the "moved" line.
  - Evidence: `027`/`028-internal-configreg` diffs.

- **F5. LOW, design.** `internal/configsync/configsync.go` (`ReconcileHubWideAt`).
  - Two error attributions have no row: `moveErr.Path` rewritten to the prime's file when the seed copy cannot move, and the `*FileError` naming the template base when its blocks cannot move.
  - The one new failure row drives `ReconcileAll` only.
  - Evidence: `029-internal-configsync.diff`.

- **F6. MEDIUM, design.** Reconcile-before-load call sites.
  - Affected sites: `internal/battencli/arm.go:300`, `internal/fabriccli/merge_verbs.go:149`, `internal/beamcli/cli.go:718`, `internal/webstercli/wiring.go:120`, `internal/shuttleparity/seats.go:180`.
  - Removing any of these calls, or moving the merge-in one after `MergeIn`, fails no test the diff adds. D5 asks for each.
  - The stale-stamp greps over these packages hit only the older batten run and beam start stamp checks (`lifecycle_integration_test.go:253`, `start_integration_test.go:106`) and the new beam land row.
  - Evidence: load-site grep and stamp-test grep.

Outside my lens, for the correctness lens: one board entry whose `run_settings` cannot move, such as a scalar at `loom.code` beside `code_max_bounces`, makes `RewriteRunSettings` fail the carry for every entry on every walk.

Tool calls:
1. Grep of added test lines outside the packages already in context, plus area-flow `Set` rows: the manifest held no tests for loomengine, configcli, the CLIs or boardcli.
2. Grep of loomengine move-table tests, hubreconcile failure hooks and the `moved` envelope: the manifest lacked those test bodies.
3. `requireMovesIntoTemplate`, stamp tests in the CLI packages, `KeyAtLine`, `FlagCodeAutoAccept` and hardencli tests: none were in context.
