# orch: host the hub orchestrator in a strand with auto handoff

Adds `lyx orch`, which runs the hub orchestrator as an interactive Claude session in a reed strand of the hub's prime worktree and cycles its context before it fills.
When context usage crosses a threshold and the session is idle, lyx has the session write a handoff, clears it, and resumes it from that handoff.
The operator keeps talking to the same pane through `lyx reed attach` or the hub workspace terminal.
Until now the remedy for a full context was a manual `/scribe:handoff` and a fresh session.

## What changes

- **`lyx orch`** (`internal/orchcli`, `internal/orchengine`) with verbs `start`, `status`, `cycle`, `stop` and the hidden daemon verb `watch`.
  Every verb refuses unless cwd is the warp prime.
  `start` is idempotent: it launches the session and the detached watcher only where missing, removes a corpse strand first, then attaches (`--no-attach` skips the handover).
  A fresh launch resumes from `--handoff <path>`, else from the last completed handoff in orch state, else from a plain start prompt.
- **The watcher** is single-instance per prime.
  It reads the run's turn-end events and context usage, and runs a persisted state machine: `idle`, `handoff-requested`, `clearing`, `resuming`.
  A restarted watcher resumes from the persisted phase without double-injecting.
  A fresh launch never replays a persisted phase.
- **`shuttleengine`** gains a provider-invariant `SessionCycler` surface and `Runner.ReadEvents`, `ContextTokens`, `SessionIdle` and `ClearSession`.
  **`claudeengine`** implements it: context tokens from the transcript, an idle-pane probe, and the `/clear` key choreography.
- Three stencils (start, handoff, resume) and an `orch` config (model, effort, threshold, idle grace, handoff timeout, poll interval) loaded degrading.
- Docs: module doc in `internal/orchengine/doc.go`, `docs/overview.md`, `CONSTRAINTS.md` (Told-Geometry bound packages, degrading config set, interactive-handoff exception for `lyx orch start`), and a roadmap entry.

## Design decisions

- **Never clear without a handoff.**
  `/clear` is typed only after the handoff file exists, is non-empty, and the handoff turn has ended.
  On timeout or a question from the session the cycle aborts to `idle` and records why.
- **Context usage is the last main-chain assistant message's** `input_tokens + cache_creation_input_tokens + cache_read_input_tokens`, sidechains excluded, read from the transcript the Stop hook names, so it follows `/clear` into the new transcript.
  An unreadable transcript yields "usage unknown", which never triggers a cycle.
- **The threshold is an absolute token count** (default 150000), since the transcript does not state the window size.
- **Idle** means the newest event is a turn end (`EventStop` or `EventWaiting`, never `EventAsk`), no newer event within the grace period (default 30s), and a fresh pane capture shows an empty input box with no turn running.
  `EventWaiting` counts because the orchestrator nearly always has background watchers outstanding.
- **Provider grammar stays in `claudeengine`.**
  `orchengine` never imports it; `/clear` is engine key grammar, while which skill writes the handoff is stencil content.
- **Handoffs are untracked** under the prime's `.lyx/orch/handoffs/` and kept, so a dirty tree never blocks `update-plugins.sh`.
- The watcher's tick-error cap (30) and watch-lock retry (5 attempts, 200ms) were not specified and were chosen during implementation.
- `orch` is excluded from the sandbox suite, since it needs a live billed session.

## To check by hand

- `TestSmokeOrch_OneFullCycle` (`-tags smoke`) compiles but was never run; it needs a live billed Claude session.
  Run it and record the outcomes the module doc currently lists as unverified:
  whether background tasks and their completion notifications survive `/clear`, and whether the session's `SendMessage` address is stable across `/clear`.
- The idle-pane fixtures in `claudeengine/testdata` are hand-written, not captured from a live pane; compare them against a real Claude Code pane.
- Known residual risks: a `SendMessage` landing between the handoff turn's end and `/clear` is lost, and on a 200k window Claude Code may auto-compact before the 150k threshold on a busy session.
