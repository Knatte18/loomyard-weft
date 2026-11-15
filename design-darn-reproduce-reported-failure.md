# Darn prompt forbids gate test, so a task from a reported test failure cannot cheaply check whether it still reproduces

`darn-reproduce-reported-failure` · Note · enhancement, darn, stencil

Imported from [issue #119](https://github.com/Knatte18/loomyard/issues/119).

What happened: a darn task's board entry asked for a decision based on a test failure seen on an older base. The darn prompt forbids running `lym gate test`, so the agent spent a long stretch tracing the shedverbs loop, keeper and arming code by hand without finding why the loop died. One run of the test, allowed afterwards by the parent, showed it already passes on the current base.

Why abnormal: the cheapest first step for a task that starts from a reported test failure is to check whether the failure still reproduces on its own base. The no-gate-test rule blocks that step, so the agent pays for manual diagnosis of a failure that no longer exists.

What to do: let a darn task that starts from a reported test failure run the named test once through `lym gate test` before diagnosing, or have the darn prompt tell the agent to ask its parent for that single run up front.

Provenance: task slug smoke-died-driver-wording; note Darn-Write.md.