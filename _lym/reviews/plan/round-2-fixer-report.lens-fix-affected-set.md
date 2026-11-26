```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fixes to F11, F12 and F13 cover every member of each finding's set.

List of findings left unchecked: none. F12 is the only finding whose premise is a set, and I reran its search. F11 is about one test. For F13, the only failure points after the save are `afterSave` and the render (board.go:186-196): `followDecisionRecords` returns `recordErr` without stopping the write, and `spawnSync`'s error is dropped. The new rollback covers both of those failure points.

**F12 — every route to a strict loom or tapestry load.** The fix covers every production site.
- Non-test files that reach `loomengine.LoadConfig`, `LoadTapestryConfig`, `configreg.SettingLookup` or `loomcli.ArmAt`: battencli/wire.go, beamcli/deps.go, fabriccli/merge_verbs.go, loomcli (arm, circling, landingdeps, prfindings, validate, wiring), shedcli/table.go, shuttleparity/seats.go and webstercli/wiring.go. Card 9 already covers each of these, through the loom or shed pre-run, batten's arm, merge-in, `hubLandDeps`, `wireHub` or `loomConfigOf`.
- `readAgentActivity` is reached only through `DobbyRunNotices` (dobbynotice.go:51), which only the dobby uses.
- Production builds the dobby's deps in one place: `runDobbyWith(..., hubDobbyDeps, ...)` at beamcli/cli.go:715. So a walk in `hubDobbyDeps` covers every dobby start and re-exec.

Finding:
- **LOW.** The text: card 9's new `hubDobbyDeps` bullet ("calls it with `prime` before it builds the dobby's deps"). The problem: many existing tier1 tests call `hubDobbyDeps` directly, and they will now run a full hub walk, but card 9 neither lists them nor says what they will see. Evidence:
  - `hubforge` removes the clone's build stamp "so a start verb driven in a test walks the hub as on a binary change" (internal/hubforge/hub.go:264, :365).
  - Direct callers include beamcli/dobbydeps_integration_test.go:60, :93, :236, :263, :289, :297, :413, :479, :565 and beamcli/dobbynoticestamp_integration_test.go:71.
  - Each of those calls now takes the hub lock, pulls the board and may commit inside the test. A test that counts board commits or creel posts could see one more. The card could add these tests to **Edit:**, or say they see no change.

Tool calls made:
1. grep for non-test files reaching the strict-load routes: rerunning F12's search, which my context lacked.
2. grep of beamcli/deps.go plus a read of battencli/wire.go:1675-1700: to check whether the `SettingLookup` at wire.go:1691 lies outside the batten verbs.
3. grep for callers of `readAgentActivity`, `gateProcessLive`, `fixerTimeout` and `hubLandDeps`: to confirm that only the dobby reaches the run-notice reads.
4. grep for callers of `hubDobbyDeps` and `hubDobbyDepsStarting`: to find every place the dobby's deps are built.
5. grep for production references to `hubDobbyDeps` and for `hubforge`'s handling of the build stamp: to confirm the one production caller and whether test hubs have a stale stamp.
