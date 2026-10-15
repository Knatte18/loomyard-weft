# Fixer report — discussion, round 1

## Changes

- F1 (fixed as proposed): D4's phase bullet now says the tokens are the persisted reading as it is, which during a non-idle phase may predate the cycle, and that the phase cell is what tells the operator a cycle is in progress.
  Premise confirmed: `tickCompacting` in `internal/orchengine/watcher.go` calls `storeCurrentReading` without a turn end, and the resume timeout sets `LastContextTokens, LastContextKnown = 0, false`.
- F2 (fixed as proposed): D2's Decision paragraph states that a present file that fails to decode is a failed read under the same tolerance, and the acceptance criterion now reads "One failed state read, a decode failure of a present file included, after a good one".
- F3 (fixed as proposed): D3's Bound line says `lyx orch start` replaces the runs strand when it is dead or its sidecar does not record it as launched by the running build.
  Premise confirmed: `ensureRunsStrand` in `internal/orchcli/runs.go` keeps the strand only when `statuspane.StrandCurrent` holds.

The support log's Turn 3 ledger line on D3's bound records that turn's finding and stays as written.

## Disputed

None.

## Deferred

None.
