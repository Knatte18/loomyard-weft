```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**

| Behaviour | Pinning test |
|---|---|
| Green record `covered`: args matching, guard `-run` filter, other tree, module or fingerprint | `TestGreenRecord` (`internal/gatecli/green_test.go:71-80`) |
| `subtract` under the `none` and `go` scopers | `TestGreenRecord` (`green_test.go:106-110`) |
| Pruning keeps 20 newest whole-green commits; malformed file replaced | `TestGreenRecord` (`green_test.go:135-196`) |
| `newestWholeAncestor` (D2 base chain: narrowed, args, fingerprint, module, non-ancestor) | `TestGreenRecord_NewestWholeAncestor` (`green_test.go:230-236`) |
| Carry takes no slot, carried run record and summary line, guard group repeat carries | `test_integration_test.go:503-514`, `:563` |
| Dirty tree reads and writes nothing | `test_integration_test.go:573-584` |
| Dirty tree diffs from the parent merge base | `scopes_integration_test.go:423` |
| Tree changing during a run records nothing (`stillGreen`) | none found |
| Launch list: cached and no-test-file packages left out | `counters_test.go:71-82` |
| `lyx gate report` grouping, carried count, launch share | `report_test.go` |
| Old record without a key or launch list in the report | not confirmed; `report_test.go:26-41` seen only in part |
| Empty-tier failure: tag, platform, `-run` exemption, `TestMain`-only | `TestEmptyTierFailure` (`emptytier_test.go:35-64`) |
| `-skip`, `-list` and `-bench` exemption | not confirmed; only `-run` seen (`emptytier_test.go:52`) |
| `card` scope refused off `card` and `batch`; `guard_tier` still loads | `internal/gateslot/config_test.go:99,125,126` |
| `MigrateConfig` for `landing`, `final`, `round` and `guard_tier` | `internal/gateslot/config_test.go:478-481` |
| `tree_scope_verify` three-way migration | `internal/darnengine/config_test.go` (not opened) |
| `--gate` refusals and `gateKeyRefusal` | `tiers_test.go` and `test_integration_test.go`, partial (`flagShapeRefusal` rows not opened) |
| Entry gate: skip, chain, empty `on_done`, over-long chain, evaluation error, resume, Stuck routed back, `pause_before`, `pause_after`, label carry | `TestStep_EntryGate` (`internal/shedengine/entry_test.go:51-300`) |
| Goto names its target as entered | `goto_test.go:48,65` |
| Relaunch clears the entered row; no-target and already-in-effect relaunch leave it | none found. `TestRelaunch` (`goto_test.go:559-678`) has no `EnteredRow` assertion. |
| Rename carries `EnteredRow` | `rename_test.go:28-40` |
| Old status without `entered_row` or `reason` reads tolerantly | `status_test.go:42,96,128` |
| `skipped` read as passed through by the history readers | `activity_test.go:40`, `rounds_test.go:87`, `classify_test.go:110`, `coherence_test.go:160`, `checklist_test.go:317` |
| Build-time check of entry-gate operands | `entrygate_test.go`, `build_test.go` |
| Non-bool bare operand is an evaluation error | not confirmed in `entrygate_test.go` |
| Batten rows skipped by entry gate; `--window` forces Open-Window | `lifecycle_integration_test.go:1289-1313`, `cli_test.go:295` |
| Darn budget 0: Bouncer approved, Describe gated off | `darnrecipe_test.go:127` |
| One-tag scan | `cmd/lyx/tiertags_test.go` (not opened) |

**Findings**

1. MEDIUM, class design. `Relaunch` entered-row semantics have no test. The clear-on-target, unchanged-without-target and unchanged-when-Admit-reports-in-effect paths are in `internal/shedengine/goto.go` (Relaunch hunks, about lines 189-245). `TestRelaunch` (`goto_test.go:559-678`) and `entry_test.go` never assert `EnteredRow` after a relaunch. D8 and the acceptance criteria require this behaviour, and reverting the `cur.EnteredRow = ""` line would break no test.

2. LOW, class design. The `stillGreen` guard in `recordPass` (`green.go`, `recordPass`) has no test that a tree changing mid-run leaves nothing recorded. This is an acceptance criterion ("a tree that changes during a run records nothing"). The integration tests I found cover only the dirty-at-start case.

3. LOW, class design. In `emptytier_test.go`, the filter exemption has a row only for `-run` (line 52). `-skip`, `-list` and `-bench` are exempt per D7 via `isFilterFlag` and `filterFlags` (`tierrun.go`) but have no row. The `=value`, `test.` prefix and two-dash spellings are also unpinned. This would be an extra row in the existing table, not a new test.

4. NIT, class consistency. `tools/tokencount` fixtures split: no lost test is visible from the stat. `calibrate_test.go` -163, `count_test.go` -157 and `fixtures_test.go` +360 are consistent with moved helpers. I did not diff the test functions.

**Test economy**

I found no duplicated new test and no per-helper test that a public-surface test already covers. `TestGreenRecord` and `TestGreenRecord_NewestWholeAncestor` are two tests over different methods, one per behaviour. The battencli `*_helper_test.go` files are splits of existing helpers, not new coverage.

**Tool calls (4 of 6)**
1. Listed changed test files per package and the `Test` function names in the new test files. The manifest held no test-file list.
2. Grepped subtest and table row names in `entry_test.go` and `green_test.go`, plus usages of `tree_scope_verify`, launch and carried in test files. The manifest held no test contents.
3. Grepped `MigrateConfig`, `OutcomeSkipped`, `EnteredRow`, `--gate` and tier-tag tests across the repo, and the `emptytier_test.go` rows. Checked whether each behaviour has a test.
4. Grepped `Relaunch` in the shed tests, carried and dirty rows in the gatecli tests, and the darn and batten test names. Checked the Relaunch gap and the integration coverage.
