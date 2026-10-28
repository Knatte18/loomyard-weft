# Batch independent calls and edits via stencil and prompt sentences

A session's cost grows with its turns times its context.
The turn audit (`docs/research/turn-audit-2026-10-28/report.md`) found that many turns of the heavy roles are single-call turns whose calls did not depend on each other, and that edits were made one per turn.
This change tells each role to batch, and tells every shuttle-launched Claude session the same in its system prompt.
Agents still read the source themselves; nothing here hands an agent pre-read or digested source.

## What changes

- Eight stencils under `contracts/stencils/` each gain a batching sentence at the step where the role first reads or edits.
  The sentences follow the report's section 5 wording with "turn" replaced by "message", the unit one batch of tool calls arrives in.
  - Darn writer, Plan and Discussion writers, twill lens directive, burler explore step, burler fixer, landing fixer and Webster implementer each carry the sentence for their role.
  - The Plan and Discussion sentence is "Issue every search and read you can name in one message; wait for a result only to choose the next call."
    The landing fixer and the twill fork boilerplate reuse it.
- `internal/shuttleengine/claudeengine` appends the sentence "Request every call whose arguments you already know in one message, and read what you need before you edit; wait for a result only to choose the next call." to the run directory's `append-system-prompt.txt`.
  The file now reads: deny notice, creel trust sentence, batching sentence, space-separated; with no deny notice it holds the two sentences alone.
  The package `init` check refuses a line break in either sentence, since the file is one line.
- `doc.go` of `claudeengine` and `docs/overview.md` describe the new file contents.

## Design decisions

- The darn writer's "test as you work" becomes one `lym gate test --wait` after the last edit of each commit.
  The stencil still asks for small commits that each leave the tree green, so the gate runs once per commit, not once per session.
  Edits within a commit go unchecked until that gate, the same bound the commit rule already sets.
- Webster's gate stays per card: the sentence asks for the card's gate once, after the card's last edit, and does not touch the per-card gate and commit that form the plan's resume trail.
- The burler stencils serve the Discussion, Plan and Code rounds alike, so one sentence each covers every burler round.
- Placement keeps the stencil markers where they were: the twill marker stays in the Plan writer's Step 2, and the burler explore step's `fork_reading` marker stays on the `tool_use_rules` line.
- The sentences live in the stencils, read at call time, and the prompt sentence stays inside `claudeengine`, per `PATTERN-stencil-ownership` and `PATTERN-shuttle-provider-seam`.
- A session opened with `lym shuttle open` writes no run directory, so it carries neither this sentence nor the creel one, as before.

## To check by hand

- The sentences may cut fewer turns than the audit's ceiling.
  No bench was run before the change; the next loom and darn runs after landing are to be measured against the report's figures by re-running its scripts on their transcripts.
- A merged message delivers its results together, which can lengthen a later context slightly.

Out of scope: a `PostToolUse` hook that nudges toward batching, to be considered only if these sentences do not cut enough.
