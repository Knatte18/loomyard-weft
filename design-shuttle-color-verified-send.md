# Shuttle: first turn on the launch line, /color through the verified send

`shuttle-color-verified-send` · Note · enhancement, shuttle

## Why

Shuttle types `/color <color>` into a fresh session's pane, and with skills it types the skill-load message and then the prompt as well.
Every typed send is a chance for a lost or half-landed message, and the operator finds the typing clumsy (2026-10-08).

## Facts (Claude Code 2.1.294 CLI reference)

- No CLI flag, settings key or environment variable sets the session color; `/color` in the session is the only way.
- The positional `claude "<text>"` submits its text as the first user turn of an interactive session, and a slash command or skill invocation works there.
- Only one positional prompt exists, so the launch line carries one first turn.

## Proposal

- The launch line carries the first turn in every case: the skill-load message when skills are named, else the prompt, so only the turns after the first are typed.
- `/color` stays typed, as there is no launch-time alternative, but goes through shuttle's verified send once `shuttle-send` lands, after the session is ready.

## Priority

Operator, 2026-10-08: low; the typed skill load is robust, and the launch line saves only a little time.
