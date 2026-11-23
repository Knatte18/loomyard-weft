---
dispositions:
  - id: F1
    disposition: fixed
---

## Changes

- F1: in `internal/loomcli/gotohook_integration_test.go` the third row's `wantErr` is now
  `could not stop the recorded PR-Review session: shuttle: stop strand guid-1: probe reed before removing it`,
  which only the built runner's failed stop yields.
  The row is renamed `PRReviewWithARecordedStrandOnAnUnwiredReceiverReturnsTheStopError`,
  and the doc comment's lines on that case name the failed probe of reed.
  Committed as `1802a5258`.
  `lym gate test ./internal/loomcli --tier 1` passed.

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
