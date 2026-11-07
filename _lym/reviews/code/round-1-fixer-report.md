# Fixer-report, round 1

## Changes

- F1 (MEDIUM), fixed as proposed, commit b6a8d42e:
  - `TestStoreHousekeeping`'s nil-probe subtest no longer calls `SetInFlightProbe`, so it runs on the default nil probe with its packed (objects/17) and prune assertions.
  - The probe table gained an "an empty answer packs the stores" row (`wantPack: true`, nil error), so the empty answer still packs.
  - The nil-probe assertion is named in the commit message.
- F2 (LOW), fixed as proposed, commit 0c03a198:
  - `TestVerbCases_ArrangeSetsItsPreconditionField` in `livestate_verbs_selftest_test.go` asserts the Arrange of Add, Checkout and Pull returns a non-empty `LauncherBlocker`, `CheckoutBranch` and `AdvancedToSHA`.
- Self-check follow-up: the comment above the probe table still named the renamed subtest as "gc-prune"; it now says "default-probe".
- Gate follow-up: the doc comment of `TestVerbCases_ArrangeSetsItsPreconditionField` wrapped at a fixed column and ran 120 characters.
  It is now one sentence per line.
  The gate then flagged the first line, still 120 characters, so that line was reworded to stay under 120.

Gate: `lym gate test` tier 0 and `--tier 1` over `./internal/fabricengine`, both green.

## Self-check

- fix-code-correctness, LOW: stale "gc-prune subtest" pointer in the probe-table comment; fixed.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
