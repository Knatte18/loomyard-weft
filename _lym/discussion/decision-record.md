## Goal

```markdown
## Seen

The landing of `discussed-payload-map-race` (`d252353e5`) records a gate failure that passed on a second run, in `internal/creelengine`:

`TestStore_MachineOutbound/WithTheSchedulerStartedAndNoTakeAMessageIsReturnedAtItsHourWithItsLastReportedReasonOrBridgeUnsettled`

That landing did not touch `internal/creelengine`, so the failure is the test's own.

## Idea

Find what the subtest waits on, likely the scheduler's hour against the test's clock, and make it wait on a signal instead of on time, so it passes every run.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-28): No test seam: the flake is a race in the test's own synchronization

Decision: The fix is test-only: post both messages before the scheduler starts, so the hour loop's first pass sees both and arms the single hour wait the test advances past. The red-first check does not apply: the subtest passes on the merge base nearly every run, and production is unaffected.

Rationale: The writer's diagnosis: a second wake can read the store before Advance(time.Hour) and the clock after it, arming a 1s floor wait the fake clock never fires. Under a real clock that wait expires a second late, so the defect lives in the test alone and has no production seam.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
