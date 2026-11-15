# Orch sessions turn on remote control by themselves, switchable per hub

`orch-auto-remote-control` · Note · feature, orch, claude, config

## Seen (operator, 2026-11-15)

The operator follows the orch from the phone, and turns on Claude Code's remote control by hand in each new orch session with `/remote-control`.
A context cycle or a fresh orch session drops it, so it must be typed again.
lym does nothing with remote control today.

## Idea

lym turns on remote control in every orch session it launches or resumes, the hub orch and numbered orchs alike.
A hub setting switches it on or off per hub, so a test hub or a work machine's hub can keep it off.
The run's sessions (writers, reviewers, beams) are not covered.

## Open

- How lym turns it on: a launch flag or setting of Claude Code if one exists, else the slash command sent into the pane after start.
- Which config file holds the switch, and its default.
- The remote session's name: the operator named this one `ly:orch`; a name with the hub in it tells hubs apart.
