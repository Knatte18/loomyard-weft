# Shuttle reads turns right: a Stop is always taken, a turn end without output holds

`shuttle-turns` · Task · bug, enhancement, shuttle · running · Webster-Burler

## Goal

Shuttle reads Claude Code's turns right: a valid Stop is always taken, a turn end without output never ends a run, and a replay corpus proves both.
Split from the run-errors bundle (operator, 2026-10-07).

## Shuttle's wait never takes a valid Stop (#411, #415)

Seen 2026-10-06 in `test-suite-prune-rest` and 2026-10-07 in `bugfix-sessions`, both at a Webster-Burler round 2.

- The burler ended its turn: `events.jsonl` held a valid Stop past `promptOffset`, with `background_tasks: []`, and both output files existed.
- `lyx shed step` never returned and had no child process; the verify gate never ran.
- A SIGQUIT goroutine dump showed the step parked in the poll loop both times:

```
shuttleengine.(*Run).Wait            internal/shuttleengine/wait.go:305 (realClock.Sleep)
shuttleengine.(*Runner).RunGated     internal/shuttleengine/run.go:643
burlerengine.(*Engine).Run           internal/burlerengine/engine.go:197
shedadapters.(*BurlerProducer).Call  internal/shedadapters/burler.go:419
shedengine.(*Shed).stepLocked        internal/shedengine/run.go:232
```

So `pollEventsTick` (`wait.go:613`) kept not returning Done for a Stop that looks complete.

- Leads: the first burler's transcript ends with a `continued-in` entry pointing at a new session, so a session switch may move the events the wait reads; an output-file or offset check is the other suspect.
  In both cases the shuttle run started a fresh strand while a round-1 strand of the same role had been used before.
- Not a suspend: the system journal shows the first case wholly awake, and the second had four minutes awake after its Stop before a hibernate (08:39:12–09:56:11), which only explains the stall's length.
- Run dirs: `.lyx/shuttle/976daeeb219e18f87d1a6954c562b64e` (first), `.lyx/shuttle/d9a47c207a52c3c94532ffc39927988b` (second).
- Wanted: find why the wait does not return, and give it a bounded exit so no SIGQUIT is needed.
  The crash-resume path after a SIGQUIT worked as designed.
- A look-alike that is not this bug: on 2026-10-07 in `bugfix-webster` an idle burler pane with a step that did not move on was a verify gate running for over ten minutes; the dump showed `evaluateGate` → `verifyrun.run` → `exec.Cmd.Wait`.
  Task `run-recovery` makes the two tell apart without a dump.

## A turn end without output holds the run instead of ending it (#407)

A turn end without every output file present ends a shuttle run as `asking`, a terminal outcome, while the agent stays alive in its pane.
Nothing waits on that agent any more: an answer sent into its session makes it work on unobserved, and resuming the producer starts a second agent on the same outputs.
Seen 2026-10-06 in `bugfix-sessions` Plan-Write: the plan agent asked, ly:orch answered in the session, the driver resumed, and shuttle spawned `plan-2` beside the still-working first agent.
The same rule makes any message to a working agent dangerous: it replies, its turn ends without outputs, and the run halts as `asking`.

- A turn end with every output file present is done, as today; a dead agent and a timeout end the run, as today.
- A turn end without the outputs holds the run with the agent alive: shuttle notifies the parent once and keeps waiting on the same agent.
- An answer to the agent is a new turn in the same session; shuttle keeps waiting.
- Resuming the producer attaches to the held run instead of starting a new agent.
- Every message to a working agent carries a fixed tail, "Answer briefly, then continue your task"; senders add it today, and `creel` adds it once it lands.

The cost: a stop no longer halts the run at once; the parent's notice and the run's timeout are the only signals.
`bugfix-sessions` (#400) already removes a live agent of the same outputs before a fresh run, which ends the duplicate but not the lost answer.

The background-shell half of #407: a turn end judged while a shell Master started still ran past `background_shell_wait_min` risks a premature stuck verdict; lyx keeps waiting on the shell or reports its final state rather than noting it may still run.

## The post-Enter redraw wait comes from `shuttle.yaml`

`submitRedrawSettle` in `internal/claudeengine/session.go`, the wait after Enter before the input box is read again, is a hardcoded 300 ms.
It comes from `shuttle.yaml` beside `submit_settle_ms`, read through `claudeengine.NewFromConfig`.

## Replay corpus and live runs

Harden shuttle against how Claude Code behaves in practice, found by running it, rather than one fix per misread found in a live run (operator, 2026-10-06).

Seen on 2026-10-06 alone: false "asking" stops at Plan-Write (twice, one spawning a second plan agent) and on a Webster master whose fork was still measuring (#407), a fork killed by an API error recorded as `dead_reason: asking`, and a burler turn end never taken at all (#411).
Each came from behaviour shuttle had not been written for: forks and background shells outliving a turn, messages arriving mid-run, skill loads, a session `continued-in` another, API errors.
Shuttle reads undocumented Claude Code internals (transcript line shapes, `custom-title`), which change between Claude Code versions.

Two parts:

- A replay corpus: every misread from a real run is saved as a fixture (the transcript and shuttle's events, with what happened), and shuttle's classification is tested against the whole corpus. Cheap and deterministic; today's cases are the first fixtures.
- Live runs: real Claude sessions driven through scripted scenarios that provoke the hard cases (long background shells, forks, a message mid-turn, skill loads, a compact, an interrupt, a question), checking shuttle's reading against the known answer. Costs tokens; catches a Claude Code release that changes behaviour before a task run meets it.

Starting point (operator, 2026-10-06): loomyard has no crucible of its own yet.
The repo's `crucible/` holds the hand-run review-and-fix loop that hardened reed, written for millhouse (`_mill/` paths, round-agent profiles copied into `.claude/agents/`), and the operator expects it can be rewritten for this more easily than starting over.
Its README names the behaviour-based form, running a live-substrate module in a sandbox, a `hardener` board note, which is no longer on the board.

What the crucible should settle is which signals tell the truth.
Shuttle already gets turn ends from Claude Code's official Stop hook, written into each run's `events.jsonl` with its `background_tasks`; the misreads come from treating "the turn ended" as "the work is done", and #411 is a Stop that arrived and was never taken.
Candidates to combine: the Notification hook Claude Code sends when it waits for input, a non-empty `background_tasks` meaning not done whatever the turn says, and quiet over time.
Codex, a planned second engine (`codex-engine`), has its own turn-complete hook with a different shape, so each engine needs its own hard cases behind shuttle's common outcomes.
