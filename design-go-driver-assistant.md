# Go driver per task, with a cold assistant and the arbiter behind it

`go-driver-assistant` · Task · feature, loom, shed

Make the Go driver (`shedrun.DriverGo`) a full alternative to the LLM loom driver: Go steps the run and settles the mechanical stops, a cold assistant is spawned only at a stop Go cannot settle, and the arbiter only when the assistant says it is needed.
The LLM driver stays as an option; the choice is per task, so the Go driver can be tested on a few tasks first.

## Evidence (agent count, 2026-10-12)

98 runs landed 2026-10-05..12 plus 3 in flight, 392 driver stops, 3.9 per run (3.1 in the newest 50):

- plain relaunch 12 %; mechanical fix a Go rule could do 52 % (relaunch after a parent decision, re-invoke under `reinvoke`, re-step on a binary change, transient retry);
- repair that needed an LLM: 0 %. No driver ever edited a file or fixed git; the action was always re-invoke or hand back;
- arbiter called 5 % (11 real attempts, all `resume`; 8 more calls refused as ineligible);
- relayed to the orch 22 % (mostly Bouncer escalations before the arbiter existed, and blocked Publish/Finalize stops whose envelope already named `goto Webster-Burler`);
- other 10 % (holds and pauses, mapping a segment name to a row, 9 `paused` stops with an empty reason that look like a shed bug).

Drivers lived 370 h, 93 % idle; in the current `--until-stop` design a driver is active about 3.5 minutes per run.
Full report kept in the orch session; rerun the count from the drive reports if needed.

## Design (operator, 2026-10-12)

Three tiers:

1. **Go driver**: steps the run with `lyx shed step --until-stop` itself and settles by rule: relaunch after a recorded parent decision, re-invoke under `reinvoke` within the repair cap, re-step on a binary change, run an envelope-named goto (capped per run), check the arbiter's eligibility before any spawn, post `parent_notice` stops to the parent.
2. **Assistant**: a cold, cheap agent (default `haiku`, configurable) spawned only at a stop no rule settles (interrupted, failed producer, unexplained stop). It reads the stop envelope, the stop report and the logs, and answers with a verdict: resume, goto, hand back to the parent, or escalate to the arbiter. It escalates when it cannot tell the cause, or judges the stop needs something stronger.
   Today's LLM driver is in effect this middle tier, but it lives for the whole run; the assistant lives only for one stop.
3. **Arbiter** (`opus[high]`): spawned by the Go driver for a Bouncer escalation it may take, or by the assistant's escalation. Unchanged otherwise.

Anything needing a parent choice still goes to the orch.

## Scope

- A per-task choice between the LLM and the Go driver through `run_settings`; the default stays the LLM driver.
- The Go driver's rules above and the assistant (model-spec, timeout, spawn budget in loom config, verdict format alike the arbiter's).
- Fix or report the empty-reason `paused` stop if it shows up in the Go path.

## Related

The notes `arbiter-when` (which stops spawn the arbiter), `judge-cap-call` (the judge settles the soft review cap) and `orch-split` (the same Go-loop-plus-cold-helper pattern at the repo level).
