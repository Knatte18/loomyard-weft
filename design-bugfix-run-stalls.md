# Runs stop stalling, and a waiting run shows what it waits on

`bugfix-run-stalls` · Task · bug, enhancement, shuttle, loom, reed

## Goal

Runs stop stalling, and a run that waits on purpose shows what it waits on.
The operator chose this bundle on 2026-10-07 as the run after `card-batching`: the "run errors" that cost the most runs and the most hub time.

## 1. Shuttle's wait never takes a valid Stop (#411, #415)

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
  Part 3 makes the two tell apart without a dump.

## 2. A turn end without output holds the run instead of ending it (#407)

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

## 3. A running verify gate is visible in status and on screen

After an agent ends its turn, a Go gate (`verifyrun`: build, unit, integration, smoke vet) can run for many minutes while the agent's pane sits idle; the operator and the orch read it as a hang (`orch-cycle-hardening` and `board-config-per-worktree` 2026-10-05, `bugfix-fabric` and `bugfix-webster` 2026-10-07, where the orch SIGQUIT-ed a gate it took for part 1).

- `lyx loom status` names the gate and how long it has run, such as `Webster-Burler: verify running 6m (go test -tags integration)`, for every producer.
  On 2026-10-07 `bugfix-sessions` showed `verify Webster-Burler gate (attempt 1, since 10:15)` in `waiting` while `bugfix-webster`, in the same producer, showed nothing: find which gate path skips it.
- On screen: shuttle owns the gate, so it sets a pane user option on the agent's strand when a gate starts (e.g. `@gate=verify`, `@gate_start=<time>`) and clears it when the gate ends; reed's status format shows it with the elapsed time computed at each refresh, such as `webster ⏳verify 2m14s` (operator, 2026-10-07).
- The same holds for every Go-side wait after an agent's turn: the Webster verify gate, the Burler round's verify, Publish's merge-in verify.

## 4. reed: stray strands after a crash (#408, #409)

- **Bare claude above a dormant orch (#408).** The folder-open chain in `internal/vscode/config.go` runs `lyx reed add --if-absent --unless-name orch --name claude`; `--unless-name` skips only while the orch strand is live (`AddStrandUnless`, `internal/reedengine/strand.go`), so after a reboot a bare `claude` strand lands above the dormant orch.
  Expected: an orch strand, live or dormant, blocks the bare add; let `--unless-name` match a persisted dormant strand, or run `lyx reed resume` before the add.
- **Resume revives finished strands (#409).** After a crash, `lyx reed resume` in `test-suite-prune-core` recreated a burler whose round was done, beside the next producer's own strand.
  A strand stays in reed's state after its work is done, since only the shuttle wait that would remove it died with the crash.
  Expected: resume brings back only strands with work left, or a strand leaves reed's state as soon as its shuttle run reaches a final outcome.

## 5. Loom status push can hit a diverged remote mid-run (#418)

Seen once in `bugfix-sessions` (drive report 20261006-121941): during Discussion-Burler round 1, loomcli logged `status push failed ... remote diverged`; the next transition pushed fine.
Distinct from #404 (Publish push).
A status push recovers from a diverged remote by itself rather than logging a failure.

## 6. The post-Enter redraw wait comes from `shuttle.yaml`

`submitRedrawSettle` in `internal/claudeengine/session.go`, the wait after Enter before the input box is read again, is a hardcoded 300 ms.
It comes from `shuttle.yaml` beside `submit_settle_ms`, read through `claudeengine.NewFromConfig`.
