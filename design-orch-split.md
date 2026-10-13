# Split the orch into a run driver and a conversational orch

`orch-split` · Note · feature, orch, batten, shed, loom

## Seen (operator, 2026-10-11)

The operator follows the orch from the phone through Remote Control, but so much comes in to the orch (batten notices, parent-review requests, escalations, PR-Gate reviews, landing routines) that the conversation is hard to follow.

## Idea (operator, 2026-10-11)

Split the hub orchestrator into two agents:

1. The run driver, the repo's upper driver: it reads the board, starts batten runs, and takes the escalated questions (parent reviews, review escalations, PR-Gate, landings).
   The operator can talk to it, but should not need to.
2. The conversational orch: the agent the operator talks to.
   It discusses problems with the operator and updates the board.

## Open

- How a question the run driver cannot settle reaches the operator: through the conversational orch, or as a notice of its own.
  The operator can keep both agents on Remote Control and gets a notification from the run driver when it needs input (operator, 2026-10-11).
- Who writes what on the board: the run driver only claims, sets statuses and prunes, while scope edits and notes stay with the conversational orch, or both write freely.
- How an operator decision reaches the run driver (start this bundle, change a hub config), and how the run driver learns the operator's settled decisions it needs at parent review.
- How much of the run driver is an LLM at all: most of the run loop is mechanical, as loom's driver now is; see `orch-owns-the-loop`.
  Starting a batten run on each ready board task (kind task, `depends_on` cleared, not claimed) needs no LLM (operator, 2026-10-11); nor does the landing routine.
  What the orch adds by hand today would need a home: per-task loom settings such as `review_max_bounces: 2` for a small task (a board field), and a cap on concurrent runs.
- `runs-overview` and `orch-token-spend` touch the same split.

## Idea: the run driver as a Go program (operator, 2026-10-12)

Perhaps the part split out of the orch is a Go-driven program: Go handles most of it on its own (starting ready tasks with their run settings, landings) and brings in the orch only when something goes wrong.
The same shape as loom today: the Go shed engine steps the run on its own (`lyx shed step --until-stop`), and the LLM driver above it wakes only when shed stops on something it cannot pass.
Per-task run settings (`config-layers-run-settings`) carry a task's review budgets and step models, so a start needs no LLM to set them.

## Two tiers, as in `go-driver-assistant` (2026-10-12)

`go-driver-assistant` (running) builds this shape for one loom run: a Go driver that settles stops by rule, a cold cheap assistant (the tackler) spawned only at a stop no rule settles, and the arbiter behind it.
The same two tiers fit the repo level:

1. A Go run driver: starts ready tasks with their `run_settings`, runs the landing routine (Land row, prod, intake), and posts what it cannot settle.
2. A spawnable assistant, cold per stop, never kept warm: it reads the stop and answers with a verdict or hands the question to the conversational orch.
   Scope and design choices always go to the orch.

Evidence from the loom level, a driver count over 98 runs (2026-10-05..12, 392 stops):

- plain relaunch 12 %; mechanical fix a Go rule could do 52 %; repair that needed an LLM 0 %;
- arbiter called 5 % (11 real attempts, all `resume`); relayed to the orch 22 %; other 10 %;
- drivers lived 370 h, 93 % of it idle.

The agent that ran the count estimated a cold helper would be needed at about 1 stop in 12 runs once Go takes the mechanical stops.
Nobody has counted the hub level yet: the orch's parent reviews, escalations, PR-Gates and landings by hand would need the same split before the Go share is known.

## A landing check for the Go run driver (operator, 2026-10-13)

At landing, loom warns that a task branch changed per-worktree config files (`_lyx/config/*.yaml`), which landing does not carry to the parent, and asks the parent to re-apply each change meant for it.
`logging-cleanup` raised it for a new, empty `_lyx/config/loom.yaml`: nothing to re-apply, and the orch found that only by diffing the file by hand.
The operator sees that check as a job for the Go run driver:

- parse the file at the fork base, the task tip and the parent tip, and compare keys;
- stay silent when the tip changes no key (an empty file, comments only) or the parent already holds the tip's value;
- otherwise name each changed key with its base, tip and parent values, so the parent knows exactly what to re-apply.

## Was `go-driven-shed`

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

## Was `run-message-routing`

Its open question is also in `go-driven-shed`'s Open above; delivery of the message stays with `creel` (triage, 2026-10-12).

Who in a running task receives an instruction from the orch or the operator, open for a brainstorm (operator, 2026-10-07).

Seen 2026-10-07: an operator decision about card 1 of `bugfix-sessions` arrived while Webster ran.
The orch has to know which agent owns the content at that moment (Discussion-Write, the plan agent, the Webster master, a Burler round) and message it, or fall back on a goto; mid-Webster no agent can change a begun card at all (`webster-amend-begun-card`).
The orch editing the run's files itself is never the answer.

Options raised:

- The driver forwards: it knows its current producer and the strand doing the work. One address per run, but an LLM relays in a turn of its own, may reword, and its stencil does not do this today.
- An alias such as `ly:<slug>:active` for the strand that owns the current step: deterministic, no LLM between.
  Creel must not know what runs or what is active (operator): it owns messages and injects only.
  So creel could at most store a plain name-to-name alias that loom itself sets at every producer transition, knowing nothing of its meaning; or the alias lives outside creel altogether.
- An on-call LLM per run, woken when an instruction needs judgment about the run (what a decision means for the plan, who must do what). The operator considered this before the driver existed, when the driver was a Go process; it rebuilds its context each time it wakes, and it adds a role.
- The driver itself becomes that on-call role, with read access to the plan: it already lives the whole run, but grows heavier than a step driver.

Plain routing ("deliver to whoever works now") and judgment ("what does this mean for the run") may want different answers.

Steps with no LLM at all: the Go producers (Preflight, Batchifier, the verify gates, Publish, Finalize, teardown) have no strand to deliver to.
A message could wait in its queue for the next LLM producer if the alias is resolved at delivery rather than at send, but that agent may not be the right one (after Batchifier comes the Webster master, while the instruction may concern the plan).
This suggests a split to discuss:
- Content ("the settle comes from config") goes into the decision record, which is durable, read by every later LLM producer, and needs no live receiver.
- Only an instruction that must act now needs a live receiver, and the most urgent one, stop, is a run pause (`lyx batten pause`), which works during a Go step too.

Operator leaning (2026-10-07): the driver takes the message.
Operator 2026-10-12: agrees that the driver takes it.
It runs a Sonnet agent on purpose, to have judgment; it is the one LLM alive for the whole run, Go steps included, it knows where the run stands, and it receives messages while its `shed step` runs in the background.
With a message it could record content with `lyx loom decision add` (a lyx verb, not a source edit), forward it to the strand that owns the current step, or pause the run or ask for a goto.

Refined (operator, 2026-10-07): every instruction for a run is posted to its driver, which gets only the short reminder (sender and subject) and does not read the body by default.
Knowing which agent works now, it either reads it itself or forwards it with `lyx creel forward` (see the `creel` note), so the receiving agent talks to the original sender, not to the driver: a minimal routing job.
