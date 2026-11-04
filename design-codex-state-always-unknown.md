# lym shuttle state reads every Codex session as unknown, which hides a running turn and an interrupt

`codex-state-always-unknown` · Note · bug, shuttle, codex

Found in the codex-trial-hooks-guards-skills trial, 2026-11-04.

## What happens

`lym shuttle state` shows `state: unknown, cause: liveness-unproven` for every Codex run it was read for, whatever the session does.
The session fold does derive the right state: the history of an interrupted run lists `idle-fresh/session-start`, `busy/turn`, `idle-stalled/interrupt` at the rollout's `turn_aborted` time, and then `unknown/liveness-unproven`.
The current state is the last entry, so a reader sees `unknown` after the interrupt, and while a turn runs.

## Cause

`SessionFold.settle` in `internal/shuttleengine/sessionstate.go` takes `facts.Liveness == LivenessUnproven` before it reaches the signal-derived state, and `codexengine.Codex.ProcessLiveness` returns unproven for every session by design (Codex keeps no session registry).
Every Codex poll therefore also logs `shuttle: session state disagrees ... stands=loop` once per disagreement.

## Expected

A Codex strand reads busy while a turn runs and idle-stalled with cause interrupt after an interrupt, as the package doc says an interrupt marks the turn interrupted. Unproven liveness need only stop the state from claiming the process is alive, not replace the turn state.

## Seen on

One interrupted run read through `lym shuttle state`; two further interrupted runs were checked in the rollout only, where each newest `turn_aborted` carries reason interrupted.