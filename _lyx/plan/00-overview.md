---
format: 5
approved: false
language: go
---

# Plan: shuttle-state

Shuttle learns to read what an agent session is doing from Claude Code's own hook reports, in shadow mode, and the stops built on today's reading stop misfiring.
The plan first declares lyx's denies in one table and adds the `python` deny, then adds the session-signal parser, keeps batten's activity reading blind to the new lines, and only then installs the new hooks and stamp lines.
On top of the signals it builds the pure state reducer, Claude's liveness and interrupt facts, the files-only reader and `lyx shuttle state`.
It then shows a held turn end as a wait, makes payload-reported background shells a wait (#429) with webster's wording kept true, and logs the state beside the wait loop's, the orch watcher's and batten's own judgments, none of which branches on it.
The corpus, an `llm`-tier capture of the new payloads and a stuck-outcome regression pin close it.

## Card Index

1 — deny-table — declare lyx's standing denies in one table that drives both the hooks and the notice
2 — python-deny — deny python in Bash for every lyx-spawned session, switched by claude_deny_python
3 — session-signals — parse hook lines into provider-neutral session signals with hook-side times
4 — activity-from-stamps — take batten's events-file activity from the newest Stop or ask stamp, not the file's mtime
5 — hook-stamps — install the UserPromptSubmit, StopFailure, Notification and SessionEnd hooks and a stamp before every recording hook's payload
6 — state-reducer — fold signals and facts into a session state, cause, since and history
7 — claude-prober — read Claude's process liveness from the session registry and the transcript's interrupt marker
8 — state-reader — read every running record's session state from files, and one run's through the Runner
9 — shuttle-state-cli — add the read-only lyx shuttle state verb and its sandbox scenario
10 — held-wait-mark — show a held turn end as a held wait that batten's quiet notice ignores
11 — payload-shell-wait — keep payload-reported background shells a wait, and finish a gated run with outputs at once
12 — webster-shell-wording — keep webster's leftover-shell warning, note and summary true for both shell kinds
13 — wait-shadow-log — log the session state and its disagreements beside the wait loop's classification
14 — orch-shadow-log — log the orch session's state beside the orch watcher's idle probe
15 — batten-shadow-log — carry and log each run's session state where batten decides a quiet or API-error notice
16 — corpus-states — state each corpus case's session states and add fork-killed-api-error's StopFailure turn end
17 — llm-hook-capture — capture real UserPromptSubmit, Notification and SessionEnd payloads in the llm tier
18 — stuck-outcome-pin — pin that a Master's stuck outcome is never reported as a question

## Shared Decisions

- Batten's activity follows the added decision "Batten activity ignores lines ParseEvents skips": an `idle_prompt` append does not move `LastActivity`.
  The acceptance row expecting it to move predates that decision and is superseded; card 4 pins the decided behavior, and card 4 lands before card 5 so no commit lets the new hooks move batten's quiet clock.
- The session signals are a separate capability (`SessionSignalParser`); `ParseEvents`, `EventKind` and every wait-loop verdict stay unchanged.
  The provider-specific facts (process liveness, the interrupt marker) come through a second optional capability (`SessionProber`), and the transcript API-error marker through the existing `ActivityReader`, so `shuttleengine` imports no provider (`PATTERN-shuttle-provider-seam`).
- The wait loop reads signals on its own cursor, which stops before a trailing unpaired stamp; `run.offset` and what `ParseEvents` reads stay today's.
- Every state-reading site only logs; a failed state read changes no decision, notice or return.
- A background shell whose recorded signal is not `SignalPayload` keeps today's `background_shell_wait_min` expiry, so existing tests built without a signal keep their meaning.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
