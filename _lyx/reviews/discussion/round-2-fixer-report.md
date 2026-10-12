# Round 2 fixer report — discussion

Every finding's premise was checked against the code: `LoadState`/`SaveState` in `internal/orchengine/state.go` take `StateLockPath` per call only, `shedrun.RunLock` and `shedrun.StatusLock` live in the scratch directory (`internal/shedrun/paths.go`), and `allowedNoTmuxMain` in `cmd/lyx/tmuxisolation_test.go` is empty.
All five hold.

## Changes

- F1 (fixed as proposed): D10 now holds the orch state write lock (`Paths.StateLockPath`) from reading the phase and `LastHandoff` through the last removal; D10's Bound adds that a handoff a cycle starts while cleanup runs is never removed; a new acceptance criterion says a cycle entering handoff-requested during `lyx orch cleanup --apply` loses no handoff.
- F2 (fixed as proposed): D9's orphan bullet defines an orphan as a scratch directory whose records run directory is absent on disk and from the records HEAD tree, whose `shedrun.RunLock` can be taken and whose newest modification is older than one day; the tracked-but-deleted bullet says such a slug's scratch is never emptied by the orphan rule; the tracked-but-deleted acceptance criterion adds that its scratch keeps its contents.
- F3 (fixed as proposed): D9's status-lock span runs from before it decides the slug is a candidate (the on-disk status read, or the records HEAD tree read for a tracked-but-deleted run directory) until the commit.
- F4 (fixed as proposed): D4's scan bullet states what an allowlist entry names, that its package's `TestMain` removes the process run directory, and what it lets through. The ledger entry sits directly after Q3 as `Q3b`, so Q4–Q10 keep their numbers.
- F5 (fixed as proposed): support-log Q5 and Interview turn 6 now name the handoff `LastHandoff` names and every lock file kept.

## Disputed

None.

## Deferred

None.
