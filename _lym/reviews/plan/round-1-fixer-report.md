# Round 1 fixer-report — plan

Every finding is fixed; F2 is fixed differently.
`lym loom validate-plan` answers `ok: true` after the fix and after the self-check's changes.

## Changes

- **F1** (fixed as proposed): card 1's `TestGateTestMapLinkedPackages` bullet has the fixture write a hand-made map for `linked` and `linked/inner` through `testcover.WriteMap` before the first call, and names the subtests this keeps passing.
  `internal/testcover#WriteMap` joins card 1's Uses.
- **F2** (fixed differently): the scoper half holds; the `go list` half is false.
  A scratch module shaped like the fixture, whose test file imports `tiergate` with no `require`, gives `go list -tags tier0,tier1,tier2,tier3 -f … ./...` exit 0, so `DeriveTargets` narrows over `TestGateTestMap`'s module.
  Card 1 now has `gateYAML` set `scoper: go` and records that the fixture narrows, so `ABaseScopeMapsThePackagesItsDiffTouches` and `ACardScopeMapsItsCardsPackages` stay as written.
  The proposed second package for the task-worktree `--base HEAD~1` call was not added.
  That call cannot narrow under any scoper: `DeriveTargets` returns whole when the worktree root holds no `go.mod` (`internal/impactset/targets.go`), and computeScope passes it the worktree root (`run.go:78`, `prebuild.go:36`), while the task worktree's module lies under `backend`.
  The second call is now a `--card 1` call under `scoper: go`, over a plan targeting `backend/solo/solo.go` and `tools/gen.go` outside the module, with a committed `other` package left unbuilt.
- **F3** (fixed as proposed): card 3 drops `ATestTheGuardKillsBesideAnotherPassesAloneAndIsListed` with its Crowded tests and adds the Intent sentence about the pressure trigger.
- **F4** (fixed as proposed): card 1's `ABadBase` row becomes `ACardWithNoPlan`.
- **F5** (fixed as proposed): card 1's package-set rule covers every whole scope and whole tier share, notes that a map request gets tier0's share alone, and says tier1's `forTier` fallback contributes nothing for a narrowed scope and repeats the modules for a `--card` scope.
  The overview's `package-set-from-scope` decision gains the whole case to match.
- **F6** (fixed as proposed, wording corrected by the self-check): card 1's Bound paragraph states the later-write-wins race and the unchecked writes until card 4.
- **F7** (fixed as proposed): `cmd/lym#testOnlyCodeAllowed` joins card 1's and card 2's Edit lists, with an Intent sentence on each, and both ImpactSummaries name the allowlist.
- **F8** (fixed as proposed): card 3 lists `AMapRunReplacesAnExistingMapWhole` and `AnEllipsisTargetMapsEveryPackageBelowIt` as wanting exit 1.
- **F9** (fixed as proposed): card 3 puts `TestMapSideBySide` and `TestMapSlotAsk` in `internal/gatecli/testmapstamp_test.go`, and its Intent widens that file's header, which no target can carry.
- **F10** (fixed as proposed): card 2's doc.go sentence drops "and GoRoot its root directory".
- **F11** (fixed as proposed): card 1 keeps `notests`, and `NamedPackageWithNoTrackedGoFileIsSkipped` wants it under `left_out` with `no tier0 or tier1 test and no map`.
- **F12** (fixed as proposed, then reworked by the self-check): the Collides stand-in no longer turns on a 2-second wait.
  The proposed `runsApart` form with a 300 ms window moved the timing margin to the side-by-side collision, so Collides is now a rendezvous with a rerun marker, as the Self-check below records.
- **F13** (fixed as proposed, second option): card 4 states that the reduction-failure path shares `checkClean`'s call inside `writeFailed` with the failed-build path and that no separate input reaches it.
- **F14** (fixed as proposed): card 1's Intent drops "and the plan's selector" from `testmapstamp_test.go`'s header.

## Self-check

- fix-consistency, LOW: card 2's ImpactSummary was stale after F7. Fixed as proposed.
- fix-consistency, LOW: card 1's ImpactSummary left out the allowlist edit. Fixed as proposed.
- fix-claims, MEDIUM: card 1's Bound sentence gave the dirty check to packages that write with none: spawn-marked-only packages, scan failures and failed builds. Fixed as proposed with the lens's wording.
- fix-proposal, MEDIUM: Collides over `runsApart` at 300 ms could miss the side-by-side overlap under load, so the D5 subtest's "both in `passed_alone`" would be timing-dependent. Fixed with my own change.
  The lens's subset-assertion option was not taken, since it would stop pinning D5's rerun-alone path.
  Instead, Collides keeps its record, passes at once when its record already exists (the alone rerun), and otherwise waits for another record, bounded by `testMapMeetWait` as `meetsAnother` is, failing as soon as one appears.
  `collidesWithAnother` is back in Create, and Uses names `testMapMeetWait`.
  Collides would time out when run alone with no earlier run, so `ATiersMemoryMBClampsItsTestsToOneAtATime` and `ATiersTestsRunBeforeTheNextTiers` now use Excludes tests, which pass when run one at a time.
- fix-proposal, LOW: Collides over `runsApart` duplicated `Excludes`. Fixed by the change above: Collides now behaves differently from Excludes.
- fix-affected-set: no findings; F2 was left unchecked by the lens.

## Disputed

None.

## Deferred

None.

## Rulings

None reached me.
