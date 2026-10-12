# lyx pause: stop every run at once, safely and on purpose

`lyx-halt` · Task · feature, batten, shuttle · running · Merge-Bouncer

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
- The step-boundary pause is not good enough and goes: it is how Millhouse had to pause, and lyx now has the means to interrupt agents directly (shuttle's session state from hooks, the reed panes, shuttle's stop verb).

## Open design points

- Resuming: whether a matching verb resumes every paused run where it stood, and how each role takes up its interrupted turn (loom's `interrupt_policy` already distinguishes `reinvoke` from `handback`).
- Scope: whether the orch session is paused too, or only runs.
- Per-run use: whether `lyx pause <slug>` pauses one run the same way, now that the boundary pause goes.
- Relation to shuttle's stop verb from `shuttle-send`, which settles a run's record before removing its strand.
- Unplanned sleep: whether lyx should also detect a wake it was not paused for (a wall-clock jump against a monotonic clock, or a systemd-logind sleep signal), resume a turn cut by a lost connection without a typed `fortsett`, and keep slept time out of idle timers and batten notices.

## Seen (2026-10-12, merged from `host-sleep-resume`)

The operator hibernated the PC overnight with `driver-arbiter` in Plan-Write.
On wake the plan writer's turn had ended with `API Error: Connection lost mid-response` (shuttle state `idle-stalled`, cause `api-error`, detail `server_error`), and the pane showed the turn as having run for 10h 51m.
Someone typed `fortsett` into the pane to resume it; the driver then spawned a fork to diagnose an interrupted stop and restarted `lyx shed step --until-stop`.
Batten then sent a `[batten notice]` reporting agents idle for almost 11 hours, counting the sleep.

In the same sleep, `config-layers-run-settings`'s Discussion-Burler-Fix lost its task prompt: the fix chair had answered only the skill-load preamble when a `tmux has-session` call hung for 10h45m and the watchdog killed it.
The driver's relaunch logged `burler: resuming a live fix` and reattached to the live strand as if the prompt had been delivered, so the chair sat held with no task until the orch sent it the path of its `prompt.md`.
A resume of a live strand should check that the task prompt, not only the preamble, reached the session.


## From issue #567

Imported from [issue #567](https://github.com/Knatte18/loomyard/issues/567).

## What happened

A loom run halted `failed` at `Discussion-Burler-Fix` with anomaly `producer-hard-failure`.
The halt note's reason is the single word `watchdog`.

The fix-chair shuttle run (`role=burler-fix-chair`, round 3) started at 2026-10-11T19:22:54Z.
Its `events.jsonl` holds only `SessionStart` and `UserPromptSubmit` stamps from 19:22:55Z and 19:22:56Z, then a `Stop` stamp at 2026-10-12T06:12:00Z, with no hook event in between.
The halt note was written at about 06:08Z, so the producer sat without any recorded activity for roughly eleven hours before the watchdog fired.

## Why it is abnormal

- The watchdog did not bound the stall in a useful time; the run held the loom for a night.
- The halt note does not say which watchdog fired, what threshold it applied or when the last activity was seen.
- The trace file for the step ends at the `shuttle: run started` line, so it records no watchdog decision either.
- After the halt, the shuttle run's `run.json` still reads `"outcome": "running"`.

## What to do

- Check which watchdog applies to a non-interactive `burler-fix-chair` shuttle run and why it took about eleven hours to trip.
- Have the halt note and the trace name the watchdog, its threshold and the last-activity time.
- Have the shuttle run's `run.json` outcome reflect the kill.

Provenance: task `config-layers-run-settings`, from note `loom-halt.md`.
