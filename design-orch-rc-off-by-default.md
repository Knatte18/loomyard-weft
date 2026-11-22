# Orch remote control is off by default

`orch-rc-off-by-default` · Task · bug, orch, config · starting · Open-Window

## What was seen (operator, 2026-11-22)

Bringing up the scribe hub's orch with `lym beam start` turned on Claude Code's remote control for it, because `internal/orchengine/template.yaml` ships `remote_control: true`.
The operator decides when a session gets remote control; lym must never turn it on by itself.

## Fix

- `remote_control` defaults to `false` in the orch template, so no hub gets remote control on any launch or resume unless its config turns it on.
- Every hub that has no explicit value takes the new default; ship the reinterpretation `PATTERN-config-layers` asks for a meaning change.
- The RC button of a pane stays the operator's way to turn it on for one agent.
- Check that no other launch path, such as a beam orch or a respawn, turns remote control on by its own default.
