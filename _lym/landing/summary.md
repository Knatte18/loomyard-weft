# Fix a race in the creelengine machine mail hour subtest

`TestStore_MachineOutbound/WithTheSchedulerStartedAndNoTakeAMessageIsReturnedAtItsHourWithItsLastReportedReasonOrBridgeUnsettled`
failed at times and passed on a rerun.
The subtest now posts both of its messages before it starts the scheduler.
Only the test changes.

The cause is in the test's own synchronization.
A post made after the start could wake another pass of `machineHourLoop`.
That pass could read the store before the test's `Advance(time.Hour)` and read the clock after it.
It then armed the `machineHourFloor` wait of one second on the fake clock, past the advance, and nothing fired it.
Neither message came back, and the wait on the first timed out.

With both posts ahead of the start, the loop's first pass sees both messages
and arms the one hour wait that the test advances past.

Production code is left as it is: on a real clock the same one-second floor wait expires on its own,
so the messages come back about a second late at worst in that interleaving.

No test reproduces the race on demand: the subtest passed nearly every run before the change,
so a passing run does not show the fix.
Check the interleaving above against `machineHourLoop`.

## Merge danger

**Door:** two-way
**Blast radius:** creelengine
