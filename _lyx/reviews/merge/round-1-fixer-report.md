# Round 1 fixer-report

## Changes

- F1 (fixed as proposed): `ParseSeedTolerant` in `internal/shedrun/seed.go` now calls `decodeSeed`, keeping the `decode: %w` wrap and the `DriverGo` default.
  `lyx gate test ./internal/shedrun` is green.
- Gate finding (PATTERN overview past its headroom line by 113 bytes): `PATTERN.md` is now 15555 bytes, under the 15564-byte line.
  The `PATTERN-pane-text-injection` index line is back to a short form (its detail already sits in its background file), and the `PATTERN-review-round` and `PATTERN-test-economy` lines are shortened.
  `lyx gate test ./cmd/lyx` is green.

## Disputed

None.

## Deferred

None.
