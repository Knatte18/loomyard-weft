```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test that pins it, from test names and case rows. I did not read the test bodies, so "pinned" means a named case exists, not that its assertion is tight.

**Pinned**
- **D2 validation** (non-empty label and run, no `#`, `%` or control character, window values, `{strand}` needs `window: agents`): `TestPlaceButtons` and the invalid-entry table in `buttons_test.go:56-66`.
- **D2 wrong-typed group or entry skipped with a Warn:** `GroupAsMappingIsSkipped`, `ScalarEntryIsSkipped`, `LabelAsListIsSkipped`, `PrimeAsMappingFallsBackToBeam` in `buttons_test.go:47-52`.
- **Prime fallback to beam:** `EmptyPrimeTakesBeam`, `NonEmptyPrimeTakesOnlyPrime`.
- **D6 placeholder filling:** `TestFillButtonPlaceholders` (`buttons_test.go:107`).
- **D5 render:**
  - `TestPinButtonRowLocked` rows cover: nil closure, closure error, shell failure, beam error, the prime never asking for the beam, the agents row, numbering after a skipped entry, task run directory and `{beam}` quoting, absent beam, confirm doubling, a second render taking a changed config, and a failing invocation.
  - The `status 3` and `status-format[2]` pins are in `windowsize_test.go:74-76`.
  - The format strings are pinned by the `statusFormatActions` entry in the bar test and the line-3 follow check in `bar_integration_test`.
- **D7 click flow:** `TestRunButtonVia` rows (`buttonclick_test.go:67-88`) cover:
  - unreadable, empty or non-JSON spec
  - range with no entry
  - strand read ok, empty or failed
  - the confirm prompt, the confirmed hash match and mismatch, and the carried strand name valid or invalid
  - `#` and `%` doubling
  - gone run directory
  - non-zero exit, spawn failure and a failed message
  - the confirmed-line quoting for both dialects (`:186`)
  - `TestRunButton_RealTmux`
- **Pinned click line, branch, shell seam, re-pin, pane option:**
  - Click line: `panebin_test.go:340-351`.
  - Binding branch and unpinned case: `bindings_test.go:104`.
  - Real-tmux click: `TestBindings_ButtonClickRunsTheVerbInTheRunDirectory`.
  - Shell seam: `TestShell_DiscardOutputAndCommandFlag`, `ChangeDirToWord` in `shell_test.go:145`, and `TestShell_Exit`. I saw no row that `ExitSuccess` is asserted on its own.
  - Stale-build re-pin: `OpLockRepinsStaleBindings` and `FailedFormatRepinIsNonFatal`.
  - `@lym_strand_name`: `spawn_test.go` rows `EmptyNameUnsetsTheStrandNameOption` and `FailedStrandNameOptionWarnsAndTheLaunchContinues`.
- **CLI verb:** the `cli_test.go` button rows cover each missing flag, confirmed without a hash, and no repository needed.
- **Config layering:** `config_test.go:113-167` covers both templates, null refused, a mapping group loading, whole-group replacement and the local layer refusing `buttons`. `desktophook/config_test.go:47-49` covers the empty and non-empty mapping refusals and the local refusal.
- **Geometry:**
  - Place for beam, task and an unparseable record: `TestReedGeometry_ButtonContext` (`hubgeom_test.go:164-166`).
  - Machine buttons told, and a refused machine layer leaving a zero node plus a Warn: `hubgeom_test.go:205` and `:228`.
  - The task's `{beam}` resolution: `geometry_integration_test` ("reed task worktree tells the beam…").

**Findings**

1. **MEDIUM, design:** The "`lym reed button` exits 0 on every path" contract (D7) has no test row for a panic, or for a `display-message` that fails on the click path.
   - Only `FailedMessageIsLoggedAtDebugOnly` exercises the failing-message path.
   - `RunButton` has no recover, and no cli test asserts exit 0 after a failure.
   - `reedcli/cli_test.go` only checks the flag-valid case, and `RunE` returns nil only because `RunButton` returns nothing.
   - Evidence: `buttonclick.go:53-59`, `reedcli/button.go:57-67`, `cli_test.go` button rows.
   - Also unpinned: the `validateButtonFlags` refusal, which sets an error exit. That contradicts "exits 0 on every path" (D7), but only for bad flags, which is plausibly acceptable.

2. **LOW, design:** The `ExitSuccess` helper and the full pinned-line shape ending `; exit 0` in pwsh have no pwsh row.
   - `panebin_test.go:340` ("button") is not shown with a pwsh variant; the `ConfirmedLineAndTmuxQuoteForBothDialects` subtest covers the confirmed line only.
   - Evidence: `panebin_test.go:340-351` against `buttonclick_test.go:186`.

3. **LOW, design:** Two D4 behaviours have no named row:
   - A failed desktop load inside the closure drops only the machine buttons while the place and values stay intact. `RefusedMachineLayerLeavesTheNodeZeroAndLogsAWarn` covers this.
   - **Not pinned:** `buttonBeam`'s fallback when `ParentOf` returns `""`, so the beam is the prime's anchor. Only the "task tells the beam" integration case is visible, and it may not cover the empty-parent path.
   - Evidence: `hubgeom/buttons.go:72-74`.

4. **LOW, design:** The `{strand}` entry's `Strand` flag after the static fill, and the repin of the status formats, are covered. But I found no test that `strandLabel`/`@strand` and `@lym_strand_name` are set together on the `panebin.go` launch path, only the `spawn_test` rows.
   - Evidence: `paneoptions.go:369-379`.

5. **LOW, design (test economy):** Possible overlap:
   - `buttonclick_integration_test`'s `TestRunButton_RealTmux` and `bindings_integration_test`'s `TestBindings_ButtonClickRunsTheVerbInTheRunDirectory` both drive the real-tmux click path.
   - The `buttons_test` invalid-entry table and the `buttonrow_test` row `InvalidEntryIsSkippedAndTheValidOneIsNumberedFromZero` overlap only on the skip.
   - I did not read the bodies, so I can't name a redundancy with certainty. No finding is raised beyond noting it.

Tool calls (3 of 6):
1. Grepped test function and case names across the new and changed test files, because my context held no test files.
2. Grepped `buttonrow_test.go` case names and the diffs of `spawn_test.go`, `bar_*_test.go`, `cli_test.go`, `lifecycle_test.go` and `geometry_integration_test.go`, because none of them had been read.
3. Grepped the `buttonrow_test.go` row names and the `status 3` / `status-format[2]` pin rows, to confirm the rows the earlier greps missed.
