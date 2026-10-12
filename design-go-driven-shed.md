# Go-driven shed with a cold LLM watcher at every level

`go-driven-shed` · Note · enhancement, shed, batten, loom, orch

## Question (operator, 2026-10-12)

How to get a Go-driven shed with an LLM watcher that steps in only when it must.

## Seen (2026-10-12)

The levels run their loops differently:

- batten: one task from Worktree-Create to Worktree-Teardown is a shed run that a Go process (`lyx batten run`) steps on its own, with no LLM; its `Run-Shed` row (engine `InnerRun`) starts the task's loom run and waits on it, and the orch gets a notice only when something stops.
- loom: the task's own shed run is stepped by `lyx shed step --until-stop`, but an LLM driver starts that command at once and waits in the background for the run to stop; the driver lives for the whole run.
- hub: the orch starts ready tasks, takes parent reviews, escalations and PR-Gates, and lands by hand where batten does not.

The operator wants the LLM involvement the same at every level.
At the hub level one LLM cannot keep a loop running for every batten run: too much for the orch to track (operator, 2026-10-12), so the hub loop must be Go, as `orch-split` proposes.

## Idea

One pattern at every level: a Go loop always runs, and an LLM watcher is spawned cold only when the loop stops on something it cannot pass.

- loom: `Run-Shed` steps the task's shed run in Go itself, instead of starting a driver that starts the loop.
  On an unexpected stop it spawns the watcher with the stop report, the way `driver-arbiter` already spawns a cold agent for a stop the driver cannot settle.
- hub: a Go loop starts ready tasks with their run settings and lands finished ones; the hub watcher wakes only on what it cannot settle (`orch-split`).
- Planned stops (parent review, PR-Gate) become cold agents Go spawns, which `alignment-agent` builds.

## Gains

- No long-lived driver session per run, which today can stall after a host sleep, run out of context or keep an old binary after a deploy (`board-field-drop`).
- Few tokens are saved: a waiting driver takes no turns.

## Open

- Who receives a message for a run when no driver lives for the whole run (`run-message-routing`): a queue the next agent reads, with lasting content going to the decision record.
- What the watcher may do: the driver's repair powers today, or fewer.
- Whether the watcher for a run is the same role at loom and hub level, with different scope.
