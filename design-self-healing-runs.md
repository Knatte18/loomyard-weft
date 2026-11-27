# Lym settles known run stops itself, so runs go on overnight without anyone pointing at them

`self-healing-runs` · Note · feature, dobby, batten, orch

## Seen (operator, 2026-11-27)

Everything is in place for autonomy, yet runs keep stopping on assorted causes,
and the operator has to tell the orch "this one is stuck, get it going".
Every LLM session runs under Go programs (dobby, batten, the driver) that see all of it,
so the known stop classes should be settled by those programs, not wait for a person or an orch turn.
Goal: lym can run overnight.

## Stops the orch settled by hand on 2026-11-27, each mechanical

- `lym loom resume` refused with `driver_not_parked` until the driver parked; the orch retried it in a loop.
- A start failed on "push lock busy" or "held board lock"; the orch ran `lym board sync` and `lym batten run` again.
- A review round ran out of budget with only findings the fixer had deferred; the orch accepted and filed a note.
- A hard cap was hit before Code-Review; the orch recorded the open findings as parent decisions and accepted.
- Idle notices for an agent that was correctly waiting on a gate loop or on an operator answer.

## Idea

- Dobby, or the driver, retries the mechanical ones itself with a bound and reports only what it could not settle.
- A rule table decides, per stop class, whether lym settles it, the orch settles it, or it goes to the operator,
  and autonomy mode's severe-halt classes are the only ones that reach the operator.
- Quiet notices name what the agent waits on, so a correct wait raises nothing.
- Collect the stop classes from the drive reports and notices of the last weeks before designing.
