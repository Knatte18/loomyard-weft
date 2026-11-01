# A card gate failure goes to a cold, cheap fixer instead of the fork

`card-gate-cold-fixer` · Note · feature, webster, gate

## Seen (orch, 2026-11-01)

A Webster fork fixes its own card gate's failures in its own session, at 150k to 450k tokens of context a turn; one fork spent 69 turns in that loop.
Webster already has a cold fixer, but only for the plan-level verify gate: a fork fixer first, then `lym webster fix-verify`, a separate process at the recovery role with a full prompt (`webster-prefix-verify-fix-cold.md`) that inherits nothing.

## Idea (operator, 2026-11-01)

A card gate failure goes to a fresh, cold, cheap fixer instead of the fork:
- Go spawns it in the fork's worktree with the gate report, the card and the card's diff; it reads the failing source itself and fixes every failure in one go.
- When it clears the gate, the fork goes on with its next card or ends.
- When it does not, the failure goes back to the fork, or up the recovery strand's ladder as today.

It rests on the gate report of note `gate-failure-report` that lets every failure be fixed at once.
