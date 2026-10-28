# Stencil and system-prompt sentences that batch independent calls and edits

`turn-batching` · Task · enhancement, stencil, shuttle · running · Code-Burler-Review

## Seen

A session's cost grows with its turns times its context.
`docs/research/turn-audit-2026-10-28/report.md` found that a large share of the heavy roles' turns are single-call turns whose calls did not depend on each other, and edits made one per turn; its section 5 words a sentence per role.
The operator agreed with its recommendations (2026-10-28); this task builds them.
Agents keep reading the source themselves: nothing here hands an agent pre-read or digested source.

## Design

- Add the report's section 5 sentence for each role to that role's stencil: the darn writer, the Webster implementer, the Discussion and Plan writers, the twill lens directive, the burler review step that reads the focus, the burler fixer and the landing fixer.
  The rule they carry: gather every file and search you can name in one message before the first edit, make independent edits in one message, and run the gate once after the last edit.
- Shuttle appends one sentence to every Claude agent's system prompt with `--append-system-prompt`: "Request every call whose arguments you already know in one message, and read what you need before you edit; wait for a result only to choose the next call."
  It sits beside the creel sentence shuttle appends, in one appended prompt.

## Measure

No bench before the change: the next loom and darn runs after landing are measured against the report's figures by re-running the report's scripts on their transcripts, as a later audit.

## Out of scope

- A `PostToolUse` hook that nudges toward batching; only if the stencil and prompt sentences do not cut enough.
