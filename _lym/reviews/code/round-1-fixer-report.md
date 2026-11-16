# Round 1 fixer-report

## Changes

- F1 (LOW): fixed as proposed. In `internal/runproc/end_test.go:524` the subtest name
  `TheHubOwnerSubjectLeavesTheHubWidePushesTestMapSlackRelayAndTeardownAlone` is now
  `TheHubOwnerSubjectLeavesTheHubWidePushesSlackRelayAndTeardownAlone`.
  Nothing else changed.
  Committed as `runproc: fix F1 — drop stale TestMap from the HubOwner subtest name`.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
