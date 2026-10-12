---
format: 5
approved: true
language: go
---

# Plan: driver-arbiter

A run's driver that reaches a stop it cannot settle runs `lyx loom arbiter`, which spawns a cold arbiter agent on a stronger model through shuttle, waits for its verdict file, and returns it.
The arbiter may record a circling decision and a design call, and names a goto target instead of moving the run.
A `resume` verdict reaches Go through the driver's re-launch, `lyx shed step --until-stop --arbiter-attempt <N>`, which checks the run against the attempt under the status lock, marks the attempt applied and performs the goto before the loop steps.
A `hand-on`, a refusal or any failure falls back to today's park and escalation, so the arbiter can only remove wakes of the parent, never lose a stop.
Batten holds its state-changed notice while an arbiter is in flight and never revives an arbiter strand; the orch reads the arbiter's verdict first when one hands on.

## Card Index

1 — shedrun-arbiter-paths — declare the arbiter's durable attempt directory and ephemeral in-flight marker paths
2 — shedengine-relaunch — add the pause-keeping, admit-checked goto the re-launch performs under the run and status locks
3 — shedverbs-arbiter-attempt-flag — give step --until-stop the attempt flag and the consumption hook it runs before the loop
4 — loom-arbiter-config — add the arbiter, arbiter_timeout_min and arbiter_max_spawns keys and their resolver
5 — loom-arbiter-records — declare the arbiter role, stop key, attempt, verdict, applied mark and marker records
6 — pending-escalation — expose whether a Bouncer run directory's latest round awaits a decision on its escalation
7 — arbiter-stencil — write and register the arbiter's opening stencil
8 — arbiter-prompt-spec — compose the arbiter's launch prompt and shuttle spec
9 — arbiter-eligibility — decide the closed list of stops the arbiter may settle
10 — arbiter-verb — implement the arbiter verb's spawn, re-run, budget, wait, timeout and verdict validation
11 — arbiter-cmd-wiring — mount lyx loom arbiter, wire its seams and commit its records
12 — relaunch-consume-attempt — consume a resume attempt through the re-launch hook
13 — decision-by-arbiter — admit the run's arbiter to lyx loom decision add --by arbiter
14 — driver-arbiter-branch — give the driver stencil its arbiter branch through a new marker
15 — batten-arbiter-hold — hold batten's state-changed notice while an arbiter is live
16 — batten-arbiter-wiring — wire the hold to the marker and keep the revive from relaunching an arbiter
17 — orch-arbiter-escalations — tell the orch an escalation now carries the arbiter's verdict

## Shared Decisions

### Decision: records-in-loomengine

The arbiter's records, their readers and writers, and the `arbiter` role constant live in `internal/loomengine`, the loom module's engine package, rather than in `internal/loomcli`.
`internal/battencli` reads the marker and matches arbiter strands, and it imports no other CLI package, the precedent `LoomDriverStrandName` set.
`internal/loomcli` still spawns the arbiter (D11); the paths stay with `internal/shedrun` (D9).

### Decision: attempt-identity

An attempt is identified by its 1-based number under the run's arbiter directory, in `attempt-<N>`.
Its directory holds `attempt.json` (Go, written before the spawn, so it counts against the budget), `prompt.md`, `verdict.yaml` (the agent's), `envelope.json` (the verb's result) and the `applied` mark.
The stop key is the status file's `current_producer`, history length and `state`.

### Decision: relaunch-line-in-envelope

The verb's `resume` envelope carries the full `relaunch` line, so neither the driver stencil nor its markers spell the attempt flag.
A refused consumption is an error envelope carrying `arbiter_attempt`, the one key the stencil branches on to take the hand-on path.

### Decision: consumption-before-the-loop

The attempt is consumed by the `--until-stop` waiter, before it reads the loop's files or spawns a loop, through `shedengine.Relaunch`.
`Relaunch` holds the run lock only for its one status write and releases it before the loop's first child takes it; a step that never comes leaves the stop key unchanged, which D7's applied-attempt refusal hands on.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
