```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Behaviours and the tests that pin them**
- `LoadRefuseRaw` returns the list when the full load fails on a gate key (scope-less `harden-live-copies`): claudeengine `TestPrepare_WritesArtifactsAndReturnsConsistentLaunch` row `ScopelessGateKeyStillLaunchesWithRefuseRaw`, and codexengine `TestPrepare_InstallsGuardsFromItsInputs` row `ScopelessGateKeyStillLaunches`. Pinned.
- `LoadRefuseRaw` with an absent file returns an empty list: claudeengine row `AbsentGateConfigRendersNoRefuseRawRow`. Pinned.
- `LoadRefuseRaw` refuses a stack that does not parse or merge: rows `UnparsableGateConfigRefusesBeforeArtifacts` and `GateConfigThatDoesNotParse`. Pinned.
- `LoadRefuseRaw` refuses an invalid `refuse_raw`, by type or by `decodeRefuseRaw`, when the full load also fails: none.
- `LoadRefuseRaw` logs the failure as a warning: none.
- `RetireHardenLiveCopies` removes and flags a scope-less entry whose tiers differ: `TestRetireHardenLiveCopies/ScopelessDifferingEntryIsRemovedAndFlagged`. Pinned.
- `RetireHardenLiveCopies` removes an equal entry, and leaves and flags a scoped entry that differs: the existing rows. Pinned.
- The flagged way forward names `--unset gates.harden-live-copies`: none. The test checks only the `--set` way forward.
- The `tier%d` in `setLive` (reinterpret.go:48, the mutation survivor): none.
- `--unset` on a worktree file syncs once: `TestDispatchSet` row "unset on a worktree file syncs once". It checks the sync count and the envelope, never that the key left the file.
- `--set` and `--unset` together are refused: `TestDispatchSet` row "set and unset are mutually exclusive". Pinned.
- `--print` and `--unset` together are refused: `TestDispatchSet` row "print and unset are mutually exclusive". Pinned.
- `--resolved` and `--unset` together are refused: none. `TestPrintResolved` has rows for `--set`, `--print` and `--layer` only (configcli_test.go:2018-2020).
- "module required with --unset": none.
- Hub-wide `--unset` of a scope-less gate key leaves a file that loads, with a sibling key kept: `TestDispatchHubWideBoard` row "unset of a scope-less gate key…". Pinned.
- Unsetting the last entry of a map drops the emptied map: `TestDispatchHubWideBoard` row "unset of the last entry of a map drops the emptied map". Pinned.
- Unsetting an absent key is refused and the file is left: `TestDispatchHubWideBoard` row "unset of a key the file does not hold…". Pinned.
- `RemoveKeys` with several keys, one missing, removes nothing: none, since every row passes a single key.
- `Unset` on an absent layer file is refused with its way forward: none.
- `--unset` rolled back when the load newly fails: none for `--unset`. The shared `setChecked` and `setHubWide` path is pinned only by `--set` rows.
- `--unset` on the local and machine layers: none, as the path is shared with `--set`.
- The Long help names `--unset`: `TestConfigLong`. This is static prose, so it is exempt.

**Findings**
1. MEDIUM, design. Location: `internal/gateslot/config.go:279-287` (`LoadRefuseRaw`'s refuse branches when the full load fails). When the full load fails on another key, nothing pins that an invalid `refuse_raw` (a bad entry, or a non-list value) still refuses the launch. Delete the `decodeRefuseRaw` or the `yaml.Unmarshal` check and every test passes, yet the summary and both engines' doc comments state this guarantee. Fix: add a row to the claudeengine and codexengine Prepare tables with a scope-less `gates` entry plus `refuse_raw: ["go  test"]` (or `refuse_raw: 3`), wanting the refusal and no artifacts. Evidence: claudeengine settings_test.go:981-985 and codexengine prepare_test.go:333-338, where no such row exists.
2. LOW, design. Location: `internal/configcli/configcli.go:209-210` (`resolvedRefusal`'s `--unset` case). The new refusal "--resolved and --unset are mutually exclusive" has no row. Fix: add a "beside --unset" row to `TestPrintResolved`. Evidence: configcli_test.go:2018-2020.
3. LOW, design. Location: `internal/configcli/configcli_test.go:573-578` (the worktree `--unset` row). The row asserts the sync count and the envelope but not the file. A worktree-layer `Unset` that wrote the wrong path, or removed nothing, would pass. Fix: assert that the seeded `discussion_timeout_min` is gone from the worktree `loom.yaml`. Evidence: configcli_test.go:596-640, where the loop reads no file.
4. LOW, design. Location: `internal/yamlengine/layers.go:30-37` and `internal/configengine/set.go:121-123` (`RemoveKeys` refusing all when any key is missing, and `Unset` refusing an absent file). Neither the "nothing is removed" rule for a list with a missing key nor the absent-file refusal is reached. Every refusal row passes one present-file key. Fix: add a hub-wide row with `unsetFlags: {"labels.old", "labels.absent"}`, wanting the file left as seeded, and an `absent: true` unset row, wanting the "does not exist" error. Evidence: configcli_test.go:1040-1050.
5. LOW, design. Location: `internal/gateslot/reinterpret.go:46-48,100-102` (the flagged way-forward text). `TestRetireHardenLiveCopies` checks only the `--set 'gates.harden-live=…'` substring. The changed way forward (`--unset gates.harden-live-copies`) and the `tier%d` from `BilledTier` (the line-48 survivor) are unpinned. Fix: also assert `lym config gate --unset gates.harden-live-copies` in the scoped flagged rows, and include `below tier` plus the billed tier's value in `wayForward`. Evidence: reinterpret_test.go:25 and :104-108.
6. LOW, design. Location: `internal/configcli/configcli.go:166-167` ("module required with --unset"). The refusal has no row. Fix: add a `TestDispatchSet` row with no args and `unsetFlags`, wanting "module required with --unset". Evidence: configcli_test.go:573-593.

The `LoadRefuseRaw` warning (config.go:288) is untested too. I file no finding for it: it is a log line, which the rubric treats as not reasonably killable.

Tool calls:
1. I read the `TestDispatchSet` loop body, grepped the configcli tests for `--unset` and `--resolved` rows, read the codexengine Prepare test loop, and grepped for `RemoveKeys` and `Unset` unit tests. My context lacked the assertions of these test loops and the `TestPrintResolved` rows.
