# Orch sessions turn on remote control by themselves, switchable per hub

`orch-auto-remote-control` · Note · feature, orch, claude, config

## Seen (operator, 2026-11-15)

The operator follows the orch from the phone, and turns on Claude Code's remote control by hand in each new orch session with `/remote-control`.
A context cycle or a fresh orch session drops it, so it must be typed again.
lym does nothing with remote control today.

## Design

Claude Code takes `--remote-control [name]` at launch, which starts the session with remote control on and names it.
lym passes that flag on every orch session it launches or resumes, the hub orch and numbered orchs alike,
when a hub setting turns it on.
The name carries the hub and the orch role, so the operator tells hubs apart on the phone.
The run's other sessions (writers, reviewers, beams' task sessions) get no flag.
A Codex orch gets nothing, as Codex has no remote control.

## Open

- Which config file holds the per-hub switch, and its default (on, as the operator uses it on every orch).
