---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
---

# Fixer report, round 1

## Changes

- F1: `strayfiles_integration_test.go` opens with `//go:build !windows`; the run-time skip and the `runtime` import are gone.
- F2: `tierResult` gained `strayLine`, set by `checkStrayFiles`; `addFailedRuns` appends a failed run labelled `runLabel(run)` carrying it.
  `TestRunTierCopiesStrayFile` runs two copies of `touch c.lock` and checks exit 1, the line once in causes, and the failed run.
- F3: `untrackedBeforeRun` takes the listing; `runGreenAware` takes it once before `retryTier` and passes it to every attempt through the new `runTierChecked`.
  `runTier` stays as a wrapper that takes its own listing, so the empty-scope call and existing callers are unchanged.
  `TestRunTierCheckedHoldsEveryAttemptToTheFirstListing` runs two attempts against one listing.
- F4: `TestRunTierStrayFilesSkipsARefusedRun` leaves a file, then starts a second invocation in a missing directory.
  Separate tests carry F2, F3 and F4 rather than table rows, because their assertions and fixtures differ from the table's.
- F5: `strayFilesLine` quotes each named path with `strconv.Quote`; the table row `ANameWithANewlineStaysOneQuotedItem` covers it.
- F6: the truncation row holds literal names `f00`..`f11` and a literal `wantNamed` of `f00`..`f09`.
- Self-check follow-ups: `doc.go`'s `# Stray files` states the once-per-tier-run listing, quoted names and the copies failed run;
  the way forward says "remove the files named here"; two doc comments were split to short lines and `runTierChecked`'s was made self-contained.
- Gate findings: the coverage map holds tier 0 tests, and none reached `runGreenAware`, `checkStrayFiles` or `strayFilesLine`.
  `strayfiles_test.go` adds tier 0 tests for each: `TestStrayFilesLine`, `TestCheckStrayFiles` and `TestRunGreenAwareFailsATierThatLeavesAStrayFile`.
  The file carries `//go:build !windows`, as the integration test does, since it shares that file's helpers.
- The fixes are in several commits, not one per finding, because the findings share files and were edited in one round.

## Self-check

- fix-code-correctness: no findings.
- fix-affected-set: no findings.
- fix-consistency, LOW, `doc.go` `# Stray files` stale: fixed.
- fix-consistency, LOW, way forward says "by their paths" beside quoted names: fixed.
- fix-proposal, LOW, over-long comment lines: fixed.
- fix-proposal, LOW, `runTierChecked` comment leans on `runTier`: fixed.

## Disputed

None.

## Deferred

None.

## Rulings

None.
