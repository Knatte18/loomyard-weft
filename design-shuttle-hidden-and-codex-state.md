# Hidden-anchor shuttle runs start, and lym shuttle state reads Codex turns

`shuttle-hidden-and-codex-state` · Task · bug, shuttle, reed, codex · draft

Two shuttle faults found in the Codex trial of hooks, shell guards and skills (`docs/research/codex-engine-trial-hooks-guards-skills-2026-11-03.md`), 2026-11-04. Fix both.

## 1. A hidden-anchor run dies before its provider is ready

`lym shuttle run --anchor hidden --model 'luna[medium]' --role x --prompt ... --output-file ...` from a worktree whose reed session is live ends with outcome died after two start attempts about 5 s apart.
The log says `shuttle: provider never became ready; strand torn down` and the error says `no pane capture was saved; the strand was removed`.
The same run with the default own-window anchor starts and finishes.

Three hidden runs, three deaths: `luna[medium]` twice and `sonnet[low]` once, so the fault is not in codexengine.
The sonnet death carried a sessionId, so Claude started a session before the strand was torn down.
Which of reed (hidden placement) or shuttle (the readiness read of a hidden pane) is at fault is not known.

Expected: a hidden strand reaches the provider as an own-window one does, or the refusal names why a hidden anchor cannot run.

## 2. lym shuttle state reads every Codex session as unknown

`lym shuttle state` shows `state: unknown, cause: liveness-unproven` for every Codex run, whatever the session does.
The session fold derives the right state: the history of an interrupted run lists `idle-fresh/session-start`, `busy/turn`, `idle-stalled/interrupt` at the rollout's `turn_aborted` time, and then `unknown/liveness-unproven`.

Cause: `SessionFold.settle` in `internal/shuttleengine/sessionstate.go` takes `facts.Liveness == LivenessUnproven` before it reaches the signal-derived state, and `codexengine.Codex.ProcessLiveness` returns unproven for every session by design (Codex keeps no session registry).
Every Codex poll therefore also logs `shuttle: session state disagrees ... stands=loop` once per disagreement.

Expected: a Codex strand reads busy while a turn runs and idle-stalled with cause interrupt after an interrupt, as the package doc says.
Unproven liveness only stops the state from claiming the process is alive; it does not replace the turn state.
