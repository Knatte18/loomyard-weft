# lym's design invariants as PATTERN entries

`pattern-design-principles` · Task · enhancement, pattern · running · Worktree-Create

## Seen (orch, 2026-11-01)

Several invariants of lym's code exist only as standing rules the orch carried in its own note, so no writer, perch or judge reads them, while every one of them reads `PATTERN.md` in full:
- A package's `doc.go` is documentation, never a home for rules.
- Fabric knows nothing about IDEs, shuttle, the board or heddle; a seam in fabric tailored to one module makes no sense.
- lym hard-codes no language, engine or branch name.
- A name or path that can go stale is never hard-coded: it is one named constant.
- What every phase machine needs belongs in shed.
- lym's own instructions to agents live in lym's code, never in heddle, which holds only repository-specific instructions.

## Idea (operator, 2026-11-01)

Each becomes a `PATTERN.md` entry, with a background file where the one line needs one, within the overview's headroom.
