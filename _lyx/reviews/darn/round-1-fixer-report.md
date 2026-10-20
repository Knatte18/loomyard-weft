# Fixer report — darn round 1

## Changes

- F1 (`913bc779`): `pattern/PATTERN-test-speed.md:42` now reads "`lyx gate test` strips the trace id from `go test`'s environment, since Go keys a cached test result on every variable a test reads".
  Fixed as proposed.
- F2 (`2964ff71`): deleted `RefreshStoredMarksAnExistingEntryUsed` from `internal/gatecli/prebuild_test.go`.
  Fixed as proposed.
  `lyx gate test ./internal/gatecli` is green.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal: LOW on F2, disputed (below).

## Disputed

- Ground `every fix worse`, fork finding on F2 (LOW): the fork notes no remaining test pins that `refreshStored` returns `(true, nil)` for an existing entry.
  If it returned `(false, nil)`, `storePrebuilt` would fall through to `copyIntoPlace`, which also stamps the mod time, so the observable result is identical and only a redundant copy happens.
  The reuse block of `TestStorePrebuilt` and the `EntryPublishedBeforeTheCallIsKept` row pin that observable result.
  Restoring the deleted test would re-add the redundancy the review's F2 removed, against `PATTERN-test-economy`.

## Deferred

None.
