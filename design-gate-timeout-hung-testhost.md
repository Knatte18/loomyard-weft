# Gate tier timeout_min does not stop a hung dotnet testhost; the slot stayed held 21 hours

`gate-timeout-hung-testhost` · Note · bug, gate, dotnet

Imported from [issue #216](https://github.com/Knatte18/loomyard/issues/216).

## What happened

In hub `dop`, run `models-torque-drag-adapter`, a Webster batch fork ran `lym gate test --tier tier3` at 2026-11-23 15:36.
Tier3 has `timeout_min: 35` in the gate config.
The tier's `dotnet test` testhost hung (about 4.5 min CPU in 21 h), and the gate slot stayed held for nearly 21 hours: `lym gate status` showed the slot held since 15:36 the next day.
The 35-minute bound never fired, so the fork, Webster and the run waited without any stop or notice to the parent orch.
Another hub's orch noticed the idle process; the parent orch killed the testhost by hand, and the gate then released its slot.

## Expected

At `timeout_min` the gate kills the tier's whole process tree (`sh -c dotnet test` → `dotnet` → `vstest` → testhost) and reports a timeout failure to its caller.

## Likely area

The timeout may kill only the direct child (`sh`), or wait on output pipes that the orphaned testhost still holds open, so the wait never returns.


## From issue #215

Imported from [issue #215](https://github.com/Knatte18/loomyard/issues/215).

On a Models hub, `lym gate test --wait --gate finalize` (PID 3536339) ran tier1, `dotnet test NORCE.Models.SlowTests -p:RunSlowTests=true` with `timeout_min: 30`, from 2026-11-23 15:36 to the next day, over 20 h. The test host had used about 7 min CPU; it was hung. The gate slot stayed held the whole time and the run sat at Finalize. On the same machine a DrillOpPlan hub's tier3 gate, started the same minute with a 35-minute bound, hung the same way and its bound never fired either. The machine may have been suspended overnight, but the bound did not fire before that either (15:36 + 30 min). Expected: the gate kills the tier's process tree at timeout_min and reports a timeout.