```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the F1, F2, F4 and F5 change set.

1. **LOW**: `_lym/landing/summary.md`, "Timings" (the table row `19.55 s | 3.65 s` and "from 29.5 s to 9.6 s").
   - The "After" figures were measured before the fix. The fix put a second `EnsureOrWarn` walk back into a test that runs serially (it captures the logger), and nobody re-measured.
   - The line-count figure in the same section was re-derived, but its sibling timings were not.
   - The fixer's gate log shows the broken-config row at 1.08s and the package at 12.236s. That run's conditions differ from the summary's (22.0s wall for the whole call), so it does not show how large the drift is.

2. **LOW**: the bodies of the F2/F4/F5 fix commit and `f2ebdb2c1`.
   - `f2ebdb2c1` says "two reinterpretation rows and three retry walks repeat another test's path".
   - F2 restored one of those three walks (the `EnsureOrWarn` broken-config row), so two remain removed.
   - `summary.md`'s "Retry walks removed" bullet was updated to match. The fix commit message corrects `f2ebdb2c1` only on which rows run alone, not on this count.

The other places I checked agree with the change:
- The 390 added / 428 removed / 38 fewer figure matches `git diff --stat`.
- The `testmain_integration_test.go` header and var comments still hold.
- "recorded and never run" at `ensure_integration_test.go:1018` still holds, since `SpawnSync` does not spawn in a test binary.

Outside this lens: `docs/research/no-fake-other-modules/fabric-batten.md:83` still names `recordBoardSyncs`. The reviewed diff made that stale before this fix.

Tool calls:
- One `grep` of the gate log `20261127T152545Z-tier1-2699253.log` for the package and `EnsureOrWarn` timings. My context showed only the `EnsureOrWarn` PASS lines, not the package time.
