# warp-visibility: LoomYard needs no CLAUDE.md of its own

`warp-visibility` · Note · feature, ide, fabric

## Seen

Using LoomYard on a repository should be invisible: nothing lym-related in the repository's own git history, and no lym text in a file the repository's own people read (operator, 2026-10-29).

## Decision (operator, 2026-10-29)

LoomYard needs no `CLAUDE.md` or `CLAUDE.local.md` of its own, in this repository or any other.
The rules agents follow are in `PATTERN.md`, which lym injects where it is needed, in the stencils lym spawns agents with, and in the skills and hooks of the lym plugins.
A session opened by hand with `lym shuttle open` has no goal and takes no PATTERN injection: it is an exploration or an interactive task with the operator, so it needs no instruction file either.
The Bolt-held `CLAUDE.local.md` copied into every worktree is dropped.

## Scope

- Check every rule of the loomyard `CLAUDE.md` against `PATTERN.md`, the stencils and the plugin skills, and move any rule that stands only in `CLAUDE.md` to the place that reaches the agents that need it.
- The cgo build requirement goes into `docs/` or an error message of the build script.
- A guard on `update-plugins.sh` that refuses a run no operator started, if the script has none, in place of the text rule.
- Delete the loomyard `CLAUDE.md`.
- `PATTERN.md` sits at the repository root and is tracked in the warp repository, so it is visible in the repository's history; the invisibility goal does not cover it yet, and it needs its own decision.
