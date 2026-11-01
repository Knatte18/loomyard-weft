# Trial: agents settle every edit before the first, then gate once

`plan-edits-before-gate` · Task · enhancement, heddle, stencil, claude · running · Darn-Write

Depends on: [`batching-in-claude-local`](design-batching-in-claude-local.md)

## Seen (orch, 2026-11-01)

The five heaviest Webster forks of the three latest tapestry runs worked in small steps: a few edits, a gate run, the next failure read, a few edits more, though each fork's card already names what to change.
That is why edits per edit turn stayed near 2.3 and gate loops ran long; `edit-with-edit-tool` gathers one file's edits into one message, and the batching sentence gathers calls whose arguments are known, but nothing asks for all of a change's edits to be settled before the first one.
A Reddit report the operator read cut agents' turns by having them plan every change before making any.

## Idea (operator and orch, 2026-11-01)

- An imperative instruction to every shuttle-launched Claude session, worded as a command, not advice: first read everything the change needs with every Read and grep call in one message, a file whole when it fits and every span of it otherwise, and a second round of reads only for files the first round named; then decide every edit across every file the card or task touches before making any; then make all of them in one message, every Edit and Write call together in one turn; then run the gate once and fix everything its report shows the same way, all fixes in one message.
- It names what it forbids: one file read per turn; one file read in slices over several turns; an edit, then a read, then another edit, spread over turns; and a gate run between edits of one change.
- It is lym's own instruction, so it lives in lym's code (`PATTERN-lym-instructions-in-code`), never in heddle: it is an entry of the read-only heddle layer lym ships, which `batching-in-claude-local` builds, aimed at the `claude-file` target beside the batching sentence (operator and orch, 2026-11-01).
- It is the one home of the read, edit and gate batching rule (operator, 2026-11-01): the same change removes the batching lines `turn-batching` put into the stencils, such as Step 4 of `darn-template-write.md` and the card steps of `webster-body-implementer.md`, keeping any card-specific part a stencil needs, such as which spans a card names; the batching sentence already left the appended system prompt with `batching-in-claude-local`.
- Checked in the transcripts of the first runs it reaches: whether the agents settle their edits before the first gate run; its effect on turns is measured with the rest of `agent-turns` by note `agent-turns-effect`.
