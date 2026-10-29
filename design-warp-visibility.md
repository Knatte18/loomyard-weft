# warp-visibility: lym leaves no instruction file in the repository, CLAUDE.local.md at most

`warp-visibility` · Note · feature, ide, fabric

## Seen

Using LoomYard on a repository should be invisible: nothing lym-related in the repository's own git history, and no lym text in a `CLAUDE.md` the repository's own people read (operator, 2026-10-29).
The text lym's agents need beyond their stencils must therefore live outside the repository's tracked files, and it holds only what lym's agents need, never general rules.

## Idea

First choice: no `CLAUDE.md` or `CLAUDE.local.md` of lym's at all.
Every rule lym's agents need goes into the stencils lym spawns them with, the skills and hooks of the lym plugins, and the system prompt text lym appends to a session, so a repository carries no lym file for them to read.
What this leaves open is what an agent reads from a `CLAUDE.md` today that no stencil, skill or appended prompt carries; find that first.

Fallback, if some rule cannot be delivered that way: one repository-wide `CLAUDE.local.md` that lives in Bolt and is copied into every worktree lym creates, beside any `CLAUDE.md` the repository already has.
Ordinary agents never write it.
It changes only through a controlled `lym` command that edits the file in Bolt, so no merging of weft branches is needed.
The copy is excluded through `.git/info/exclude` by `fabricengine`, never a tracked `.gitignore` (the Fabric Git entry, `PATTERN-fabric-git`).

## Open

- How Claude Code loads `CLAUDE.md`: a file in a worktree reaches the agents started inside that worktree, anchor subdirectory included, and no agent of another worktree.
  Check this against the Claude Code documentation before relying on it.
- `PATTERN.md` sits at the repository root and is tracked in the warp repository, so it is visible in the repository's history; the invisibility goal does not cover it yet, and it needs its own decision.
