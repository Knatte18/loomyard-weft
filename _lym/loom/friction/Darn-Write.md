# Diagnosing a smoke failure under the no-gate-test rule

The board entry asked for a decision based on a failure seen on an older base.
The darn prompt forbids running `lym gate test`, so I spent a long stretch tracing the shedverbs loop, keeper and arming code by hand without finding why the loop died.
One run, allowed by the parent, showed that the test already passes on the current base.
A darn task that starts from a reported test failure would save that time by checking first whether the failure still reproduces on its own base.
