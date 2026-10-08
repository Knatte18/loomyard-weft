# lyx pause: stop every run at once, safely and on purpose

`lyx-halt` · Note · feature, batten

## Why

The operator suspends the laptop while runs are in flight.
A sleep can cut a network call or an agent turn midway (a weft push hung a batten run for 2.5 hours after a hibernate, #458), and after resume idle notices fire for time that was only sleep.

The existing `lyx batten pause` and `lyx loom pause` only set a request the machine consumes at its next producer boundary: they kill nothing, so an agent turn, a Webster batch or a review round runs on until it ends, which can be an hour.
That is not enough to make the hub safe to suspend.

## Proposal (operator, 2026-10-08)

- One hub-wide verb, named `lyx pause` (the operator's choice of name), that safely stops every lyx process within a short time.
- It gives running work a few seconds to finish cleanly, such as a bash command completing.
- It interrupts every running Claude Code instance by sending Esc.
- A pause is ordered and intentional: it must not trigger the paths for an unexpected stop, such as batten notices, the agents-idle or driver-dead conditions, review-round failures or crash recovery.

## Open design points

- Resuming: whether a matching verb resumes every paused run where it stood, and how each role takes up its interrupted turn (loom's `interrupt_policy` already distinguishes `reinvoke` from `handback`).
- Scope: whether the orch session is paused too, or only runs.
- The existing boundary pauses: whether they stay as they are beside the new verb, are renamed, or become a mode of it.
- Relation to shuttle's stop verb from `shuttle-send`, which settles a run's record before removing its strand.
