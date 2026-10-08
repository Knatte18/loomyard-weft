# lyx halt: stop every run safely and on purpose

`lyx-halt` · Note · feature, batten

## Why

The operator suspends the laptop while runs are in flight.
A sleep can cut a network call or an agent turn midway (a weft push hung a batten run for 2.5 hours after a hibernate, #458), and after resume idle notices fire for time that was only sleep.

## Proposal (operator, 2026-10-08)

- One hub-wide verb, `lyx halt`, that safely stops every lyx process within a short time.
- It gives running work a few seconds to finish cleanly, such as a bash command completing.
- It interrupts every running Claude Code instance by sending Esc.
- A halt is ordered and intentional: it must not trigger the paths for an unexpected stop, such as batten notices, the agents-idle or driver-dead conditions, review-round failures or crash recovery.

## Open design points

- Resuming: whether a matching verb resumes every halted run where it stood, and how each role takes up its interrupted turn (loom's `interrupt_policy` already distinguishes `reinvoke` from `handback`).
- Scope: whether the orch session is halted too, or only runs.
- Relation to `batten pause`, which holds one run, and to shuttle's stop verb from `shuttle-send`, which settles a run's record before removing its strand.
