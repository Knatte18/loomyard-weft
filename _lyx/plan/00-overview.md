---
format: 5
approved: false
language: go
---

# Plan: pause-esc-rewind

`lyx pause` sends Esc only into a pane that shows a running turn, and spaces any second Esc by the provider's settle time, so it never opens Claude Code's rewind picker.
A session whose turn ended with background work is reported under `background` instead of being interrupted.
The plan builds bottom-up:

- the running-turn predicate on the session state;
- the provider's pane readings for a running turn and the rewind picker, with the settle time;
- the guarded interrupt and the guarded picker close in shuttle;
- Inject's refusal of the interrupt key and the PATTERN entry;
- the busy reason as data;
- the CLI help;
- the pause and resume sequence;
- the orch watcher's cross-tick retry.

## Card Index

1 — running-turn-predicate — add `SessionState.RunningTurn`, shared by send readiness and the pause
2 — pane-readings — declare the `InterruptGuard` and `RewindPickerReader` capabilities and implement them in Claude, with the captured picker fixture
3 — shuttlefake-guard — let shuttlefake script the running-turn and picker readings and the settle
4 — guarded-interrupt — guard every interrupt key by the pane and close an open rewind picker once, in interrupt and send
5 — inject-refusal-and-pattern — Inject refuses the interrupt key; add `PATTERN-guarded-interrupt` within the overview's headroom
6 — session-busy-reason — carry a held send's readiness reason in `SessionBusyError`
7 — shuttle-cli-guard — `lyx shuttle interrupt` help, and a CLI test of both verbs' guard failures
8 — pause-sequence — the pause interrupts only a running turn, settles before a guarded retry, lists background work, and resume reports the guard's reason
9 — orch-watcher-settle — the orch watcher waits the settle across ticks before a guarded second Esc and logs refusals

## Shared Decisions

- Provider specifics stay in `claudeengine`: `shuttleengine`, `pauseengine` and `orchengine` see only `InterruptGuard`, `RewindPickerReader`, `Runner.InterruptSettle` and `Runner.PaneRunningTurn`.
- `ErrNoRunningTurn` means "nothing to stop", and a caller that only needs the turn stopped (the pause and the orch watcher) treats it as success.
  `ErrRewindPickerOpen` is a failure and never wraps `ErrSessionBusy`.
- The pause record's shape is unchanged; every new list lives in the report.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
