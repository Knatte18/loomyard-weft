# warp-visibility: LoomYard adds no instruction file to a repository

`warp-visibility` · Note · feature, ide, fabric

## Seen

Using LoomYard on a repository should be invisible: nothing lym-related in the repository's own git history, and no lym text in a file the repository's own people read (operator, 2026-10-29).

## Decision (operator, 2026-10-29)

LoomYard adds no `CLAUDE.md` or `CLAUDE.local.md` of its own to any repository.
What lym's agents need comes from `PATTERN.md`, which lym injects where it is needed, from the stencils lym spawns them with, and from the skills and hooks of the lym plugins.
A repository's own `CLAUDE.md`, if it has one, is its team's and is read as usual.
A session opened by hand with `lym shuttle open` has no goal and takes no PATTERN injection: it is an exploration or an interactive task with the operator, so it needs no instruction file either.
The Bolt-held `CLAUDE.local.md` copied into every worktree is dropped.

## Open

Where a repository's own rules for lym's agents live, out of sight of the team.
`pattern-init` puts `PATTERN.md` at the repository root, tracked in the warp repository, so it shows in the repository's history; the choices are a `PATTERN.md` held in weft, or a review lens in Bolt such as `lenses.yaml`.
It needs its own decision before a task is drafted.
