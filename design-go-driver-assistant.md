# Go driver per task, with the tackler (cold assistant) and the arbiter behind it

`go-driver-assistant` · Task · feature, loom, shed · running · Plan-Write

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
   It is spawned cold at each stop, never kept warm on standby (operator, 2026-10-12): a standby session knows nothing of a stop before it happens, so it would save only the process start, while it would have to live, be watched and could die or hang for the whole run.
   Its name is **tackler** (operator, 2026-10-12): in the Lancashire weaving sheds the tackler was the mechanic called when a loom stopped, who fixed it and set it running again. The name fits any shed run, not only loom.
3. **Arbiter** (`opus[high]`): spawned by the Go driver for a Bouncer escalation it may take, or by the assistant's escalation. Unchanged otherwise.

Anything needing a parent choice still goes to the orch.

## When the arbiter is spawned

1. **By the Go driver directly**, with no assistant: a Bouncer escalation at the hard review cap, or a blocker that returns (see `judge-cap-call`). That is a question about content, the arbiter's job.
2. **On the assistant's `escalate` verdict**, when it cannot name the cause from the envelope, the stop report and the logs, or when the cause needs code read and judged (is a review finding right, is a fix sound).
3. **Automatically by the Go driver** when the same stop key returns after an assistant `resume`: the assistant was wrong, so the stop goes one tier up.

A scope or design choice never goes to the arbiter: it goes to the orch.
`arbiter_max_spawns` per run stays the bound; with the budget spent, the stop goes to the orch.
The arbiter answers `resume`, `goto` or `hand-on`; the Go driver performs it, and `hand-on` goes to the orch.

## The assistant's case brief

On `escalate` the assistant does the legwork and writes a case brief the arbiter receives (operator, 2026-10-12), so opus time is not spent searching.
The brief has fixed parts:

- the question the arbiter must answer;
- facts, each with a pointer (file and line, log path and line, trace id);
- what the assistant ruled out, and how;
- its hypothesis, marked as a hypothesis.

The arbiter checks the facts it relies on itself and never takes the hypothesis as given, so a wrong cheap diagnosis cannot steer the verdict.
The assistant writes only the brief: the arbiter's instructions, its allowed actions and its verdict format come from lyx's own stencil, so no LLM changes what the arbiter may do.

## Scope

- A per-task choice between the LLM and the Go driver through `run_settings`; the default stays the LLM driver.
- The Go driver's rules above and the assistant (model-spec, timeout, spawn budget in loom config, verdict format alike the arbiter's).
- Fix or report the empty-reason `paused` stop if it shows up in the Go path.

## Related

The notes `arbiter-when` (which stops spawn the arbiter), `judge-cap-call` (the judge settles the soft review cap) and `orch-split` (the same Go-loop-plus-cold-helper pattern at the repo level).
