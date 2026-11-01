# Trial: agents settle every edit before the first, then gate once

`plan-edits-before-gate` · Task · enhancement, heddle, stencil, claude

Depends on: [`batching-in-claude-local`](design-batching-in-claude-local.md)

## Seen (orch, 2026-11-01)

The five heaviest Webster forks of the three latest tapestry runs worked in small steps: a few edits, a gate run, the next failure read, a few edits more, though each fork's card already names what to change.
That is why edits per edit turn stayed near 2.3 and gate loops ran long; `edit-with-edit-tool` gathers one file's edits into one message, and the batching sentence gathers calls whose arguments are known, but nothing asks for all of a change's edits to be settled before the first one.
A Reddit report the operator read cut agents' turns by having them plan every change before making any.

## Idea (operator and orch, 2026-11-01)

- An imperative instruction to every shuttle-launched Claude session: read what the change needs, decide every edit across every file the card or task touches, make them all, then run the gate once and fix everything its report shows in one round.
- It is lym's own instruction, so it lives in lym's code (`PATTERN-lym-instructions-in-code`), never in heddle: it goes through the CLAUDE-file channel `batching-in-claude-local` builds in claudeengine, beside the batching sentence (orch, 2026-11-01).
- Checked in the transcripts of the first runs it reaches: whether the agents settle their edits before the first gate run; its effect on turns is measured with the rest of `agent-turns` by note `agent-turns-effect`.
