# Shuttle: one session-state model, held turns as waits, stuck not a question, a deliberate permission setup

`shuttle-state` · Task · bug, enhancement, shuttle · awaiting · PR-Gate

Shuttle reads what an agent session is doing from what Claude Code reports, and the stops built on that reading stop misfiring.

## One session-state model

Decided in direction by the operator 2026-10-07: unify how lyx knows what an agent session is doing, owned by shuttle.

## Why

Every module guesses an agent's state its own way, and each guess has failed:

- Batten judges a run quiet from the child's status file, which changes only at a step's end; on 2026-10-07 an API error stalled all five runs and the quiet notices came 45 min later, long after the agents were resumed by hand, naming no cause.
- The orch watcher runs its own idle probe and reads turn ends, and its compact cycle missed its own compaction (`orch-compact-timeout`).
- Shuttle's turn reading misses Stops and waits on finished background shells (#411, #420; `shuttle-turns`).
- An injected text races the operator's typing between the idle probe and the Enter (see `creel`).
- Nothing notices a live session stopped by an API error, machine sleep or a dropped network; elapsed time and transcript growth cannot tell it apart from a long simulation or test run.

## Direction

Shuttle owns one session-state model per agent strand, and batten, the orch watcher, creel and reed read it instead of inferring their own:

- `busy`: a turn is running, however long; a long tool call or simulation is never mistaken for a stall.
- `idle-done`: the turn ended with its output.
- `idle-stalled`: the session is idle but its step is not done, or the turn ended on an API error or an interrupt; detected in seconds from state, not from a timeout.
- `asking`: the agent waits on an answer.
- `dead`: the pane or process is gone.

Each state change is an event with a timestamp, so a consumer reacts to the change and keeps a history of changes only, never ticks (see `orch-notice-channel`).
`idle-stalled` gets a bounded resume: shuttle tells the session to continue, with a retry cap and backoff, then halts the run visibly with the cause.

## Sources, never guesses

Today's misreads all came from inference: scraping the pane (the idle probe, the paste race), scanning the transcript heuristically (missed `queued_command` lines, #411/#420), and judging from mtime or elapsed time (the 45 min quiet notice).
The model rests on what Claude Code itself reports:

- Hooks are the truth: `UserPromptSubmit` is a turn start, `Stop` a turn end (its payload carries `background_tasks`), `Notification` an agent asking or waiting on a permission. Shuttle already installs the Stop hook; the others are a few lines in the same hook config writing to the same event file. Hooks are only for agent sessions; lyx's own Go processes know their state already.
- An API error is a marked transcript entry; whether Claude Code also has a hook for it is to be checked.
- Liveness is the process: a dead pid is `dead`.
- The pane is a cross-check only, never the basis.
- Missing or contradicting signals give `unknown`, shown, never a silent guess.

Tests replay the real transcripts and hook payloads that today's and yesterday's failures left behind as fixtures.

## Built in small steps

A big-bang rewrite of shuttle is the risk; nothing changes behavior until it is proven:

1. `shuttle-turns` lands first.
2. Shadow mode: shuttle computes and logs the state beside today's guesses, with no consumer reading it; disagreements are read against real runs.
3. One consumer moves at a time, each its own small task: batten's quiet notice first (worst case a wrong notice), then the orch watcher, then creel.

## Relations

`shuttle-turns` fixes turn reading and is a first piece of this; this note is the step after it lands.
`run-recovery` covers dead processes; `orch-notice-channel` and `orch-compact-timeout` are consumers that would read this state; `creel` stage 2 delivers on it.


## A held turn end shows as a wait

From run-recovery PR-Gate (2026-10-07): run-recovery made every wait visible (reed `@lyx_wait` mark, `waits` token, `lyx loom status` waiting line) for verify, gate and background-shell waits. shuttle-turns adds a new hold state, a turn end without output that holds; unless it sets a wait mark too, a held session looks like a hang. Wire the hold into the same marker files once both have landed.
Also from shuttle-turns PR-Gate (2026-10-07): a gated run whose last event is a waiting turn end with every output file present can sit up to `background_shell_wait_min` (10 min) before it finishes Done.


## A stuck outcome is not a question

Seen 2026-10-07 in card-batching: the batten notice and loom status said `webster master is asking a question`, but the master had written `_lyx/webster/outcome.yaml` with `outcome: stuck` plus `summary.md` and ended its turn. The stop must be classified from the outcome file, not as a question; see `session-state` for the wider model.

## From issue #429

Imported from [issue #429](https://github.com/Knatte18/loomyard/issues/429).

**What happened**

During the Webster row, Master delegated a batch to a fork that ran a long after-state measurement in a background shell.
Master's turn ended with only interim fork output, because it was holding `record-batch` until the fork's final completion notification.
lyx's wait ran past `background_shell_wait_min` (10 minutes), stopped waiting and counted Master's turn end.
It did not stop the shell.
The run's outcome became `error: webster: master asked a question instead of finishing`, and the loom halted `blocked` with anomaly `escalation-to-human`.

Friction notes reported "Master's wait counted a turn end past a background shell" twice, in two different Master sessions.
The driver re-stepped the Webster row after a binary change (`last_step.binary_changed`) and Master blocked the same way a second time, still waiting on the same fork's measurement.
The halt recurred, so one re-step does not clear it.

**Why this is abnormal**

An interim "still waiting on a background job" turn end is not a question to a human, and halting the loom as `escalation-to-human` for it needs an operator for no reason.
The wait gives up on the shell after a fixed time, but the shell and the Master strand stay alive until the next `lyx webster run` reclaims them, so the work being waited on is killed or orphaned on the very re-step meant to recover.
The log line "stopping a leftover live strand before respawning it; an in-flight batch run restarts from state.json rather than resuming" shows the re-step discards the pending wait.

**What to do**

- Do not classify a Master turn end that is waiting on a live fork or background shell as a question or an `error`.
- Either keep waiting while the fork's background job is live, or resume Master when the job completes, instead of halting.
- If giving up after `background_shell_wait_min` stays, stop the shell and surface a distinct anomaly, not `escalation-to-human`.

Provenance: task `test-tiers`; notes `webster-background-shell.md`, `webster-background-shell-2.md`, `loom-halt.md`, `loom-halt-2.md`, `driver-restep-20261007-webster-blocked.md`.


## A deliberate permission setup

Operator 2026-10-03, while striking the roadmap's shuttle generic tools-restriction: restricting tools per agent is meaningless today, but lyx needs a correct setup of denied and allowed tools and commands, so nothing gets blocked.
Seen the same day: the global deny list (`git reset --hard`, removed by the operator; `git push --force` still in) stopped `pair-lifecycle` twice at steps lyx itself prescribed.
Related: `no-denied-recovery`, `orch-live-hardening` (permission modes mean what they say).

Seen 2026-10-04 in `board-model` (#361): Webster Master ran `python3 - …` with a misplaced heredoc, the shell hung on stdin and blocked the verify gate for 75 minutes.
Operator: lyx agents never use Python. Nothing denies it today: agents run in bypass, and shuttle's PreToolUse hook (`internal/shuttleengine/claudeengine/settings.go`) guards only other cases. Want: the same hook denies `python`/`python3` in Bash for every lyx-spawned agent, with a reason pointing to Edit/Write and grep/awk.
