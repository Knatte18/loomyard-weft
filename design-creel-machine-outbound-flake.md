# A creelengine scheduler test fails at times and passes on a rerun

`creel-machine-outbound-flake` · Task · bug, creel · done

## Seen

The landing of `discussed-payload-map-race` (`d252353e5`) records a gate failure that passed on a second run, in `internal/creelengine`:

`TestStore_MachineOutbound/WithTheSchedulerStartedAndNoTakeAMessageIsReturnedAtItsHourWithItsLastReportedReasonOrBridgeUnsettled`

That landing did not touch `internal/creelengine`, so the failure is the test's own.

## Idea

Find what the subtest waits on, likely the scheduler's hour against the test's clock, and make it wait on a signal instead of on time, so it passes every run.
