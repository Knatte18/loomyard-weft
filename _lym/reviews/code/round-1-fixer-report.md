# Round 1 fixer-report

## Changes

- **F1** (commit `72b40513`): fixed with my own change.
  - Proposed and not made: a row in `TestPushFailureClasses` that installs a fake with `InstallPushNotifier`.
    That test is parallel and cannot install the process notifier, and the review's own fix text asks for a serial test.
  - Made instead: `TestSyncOptionsNotifier` in `internal/fabricengine/remoteteardown_test.go`, a serial tier 0 test.
    One row shows a nil `Notifier` mailing through the installed notifier.
    The other shows a set `Notifier` mailing only itself.
- **F2** (commit `c4fbc3f9`): fixed as proposed.
  `GCAutoThresholdForTest` in `export_test.go`, and a `TestStoreHousekeeping` subtest asserting `NewTopology` starts at 6700.
- Doc comment of `TestStoreHousekeeping` now mentions the new subtest, after the fix-consistency lens (uncommitted at the time of writing, committed with this report's round).

The gate (`lym gate test ./internal/fabricengine --tier 0,1`) passed before the self-check, with no failures.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, `TestStoreHousekeeping` doc comment did not mention the new subtest; fixed as proposed.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
