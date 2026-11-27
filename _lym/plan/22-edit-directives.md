# Card 22 — edit-directives

**Prosa:**
- `contracts/stencils/engine/claude/edit-directive.md`
- `contracts/stencils/engine/codex/edit-directive.md`

**Intent:** D10: both engines' edit directives say that an agent may push its own worktree's commits, that a failed push needs no action, and that lym pushes its commits later.
The line goes after line 22 of each file, which keeps its rule on other worktrees, in each engine's own wording; both files stay marker-free.
No row or gate requires an agent's commits on origin: no production file under `internal/gateslot`, `internal/gatecli`, `internal/cardverify` or `internal/verifytree` reads origin, so no code changes.
The wording is checked by review.
