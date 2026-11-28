# Post the machine hour subtest's messages before the scheduler starts

`TestStore_MachineOutbound/WithTheSchedulerStartedAndNoTakeAMessageIsReturnedAtItsHourWithItsLastReportedReasonOrBridgeUnsettled` failed at times and passed on a rerun.

## Cause

The subtest started the scheduler and then posted two messages to another machine.
Each post wakes the machine hour loop for a pass, and the subtest advanced the fake clock by an hour once it saw one hour wait armed.
That wait could come from the pass the first post woke while the pass the second post woke was still running.
When that second pass read the store before the advance and read the clock after it, the loop armed its one-second floor wait past the hour.
Nothing advanced the clock again, so neither message came back and the wait on the first message timed out.
Under a real clock the floor wait expires a second later, so the scheduler itself is correct, and the change is test-only.

## Change

The subtest now posts both messages and reports the hold before it starts the scheduler.
With no hook installed yet, the posts wake nothing.
The loop's first pass then sees both messages and arms the one hour wait the advance fires.
A comment above the posts states why they come first.
The other subtests still cover a post after the start waking the loop.

## Design decisions

- No production change: the race lies in the test's synchronization, and no test can reach it through the scheduler's code, so the test that fixes it passes on the merge base as well.

## Merge danger

**Door:** two-way
**Blast radius:** creelengine
