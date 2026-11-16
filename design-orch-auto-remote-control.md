# Orch sessions turn on remote control by themselves, switchable per hub

`orch-auto-remote-control` · Task · feature, orch, claude, config, reed · running · Seed-Child

## Seen (operator, 2026-11-15)

The operator follows the orch from the phone, and turns on Claude Code's remote control by hand in each new orch session with `/remote-control`.
A context cycle or a fresh orch session drops it, so it must be typed again.
lym does nothing with remote control today.

## Design (operator, 2026-11-16)

- At launch.
  Claude Code takes `--remote-control [name]`, which starts the session with remote control on and names it.
  lym passes that flag on every orch session it launches or resumes, the hub orch and numbered orchs alike, when a per-hub setting turns it on.
  The run's other sessions (writers, reviewers, task sessions) get no flag; a Codex orch gets nothing, as Codex has no remote control.
- The name.
  The remote session's name carries the machine, the hub and the orch role,
  so the same hub's orch on two machines, or two hubs on one machine, never share a name on the phone.
  The name comes from a template in the same config as the switch, with placeholders such as `{machine}`, `{hub}` and `{role}`,
  so the operator changes the form in config alone, never in code.
  The default template is lym's cross-machine name form, `<machine>::<name>` from `agentname.FormatMachineAddress` (`neve::ly:orch`), so the phone shows the name creel uses; the operator may still choose another form, such as `ly:orch[neve]`.
- A reed button.
  An `RC` button on the orch session's status bar turns remote control on in a session running without it, by sending `/remote-control <name>` into the pane, with the same name as at launch.
  The button shows its state by colour: lit while remote control is on, dim while off.

## Open

- Which config file holds the per-hub switch and the name template, and their defaults (the switch on, as the operator uses it on every orch).
- Where lym reads whether remote control is on, for the button's colour: a launch with the flag or a click sets it, but the operator can also turn it off inside Claude Code.
  Find a signal Claude Code gives (transcript, status line payload, session files); if none, the colour shows what lym last set.
- Whether reed's status-bar buttons can change colour today, or need that added.
