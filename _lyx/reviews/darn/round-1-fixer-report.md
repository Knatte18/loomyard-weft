## Changes

Both changes are in `cmd/lyx/csharpfixtures_integration_test.go`, committed together as `0d0b4f9ee`.
Neither finding needed a new test: F1 is a list drift and F2 is a skip path no hermetic test can reach, so each is corrected in the existing test.

- F1, fixed as proposed: `csharpFixtureSkipDirs` gained `_mill`, `.wiki` and `_raddle`, matching `skipDirNames` in `internal/testkit/scankit/scankit.go`.
- F2, fixed as proposed: `dotnetTargetFramework` calls `t.Skipf` when `dotnet --version` fails, so a host with the runtime but no SDK skips.
  `Fatalf` stays for output with no dotted version.
  The doc comments of the test and of `dotnetTargetFramework` now state the skip.

Both fixes went into one commit rather than one per finding, because they touch one file and were edited before the first commit.

`go vet -tags integration ./cmd/lyx` is clean, and `lyx gate test --upto 2 ./cmd/lyx` passed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
