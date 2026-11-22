# Let hardener rounds run fabric verbs in their own sandbox hubs

The fabric command's sandboxed-role guard refused every mutating `lym fabric` verb to a task session's strand, judged on the strand name alone.
A hardener round drives its throwaway hubs from such a strand, so the guard also refused add, push, pull, sync, reconcile, checkout, remove, clone and cleanup `--apply` inside the round's own sandbox hub.
Round 1 of `harden-fabricengine` left seven of its twelve charter items unverified for that reason.

## What changes

- `sandboxedRoleRefusal` (`internal/fabriccli/roleguard.go`) now also passes a verb whose directory lies at or under the sandbox root of the strand's own worktree, `<worktree>/.lym/harden-sandbox`.
  The directory is the cwd the invocation carries, and for `clone` the destination its `--into` names.
- `inOwnSandboxRoot` resolves symlinks in that directory, takes the nearest enclosing `.lym/harden-sandbox`, and requires the directory holding that `.lym` to resolve through `lymcwd.ResolveWorktree` as a worktree root named the strand's slug.
  A verb aimed at the real hub, at another worktree's sandbox root, or at a lookalike directory that is no worktree stays refused.
- The refusal text now names the sandbox root as the place a task session may run the verb.
- `clone`'s `--into` resolution moves into `cloneDestination`, which both the clone verb and the guard call, so the guard judges the directory the clone writes to.
- The package doc of `internal/fabriccli` gains a section on the guard, and the hardener review stencil tells the round agent where the verbs run.

## Design decisions

- The pass is open to every task role, not only the hardener round roles: a sandbox root exists only in a hardener run's worktree, and the guard stops mistaken calls, not a session that builds a sandbox root of its own to slip past it.
- The sandbox root is joined from the `lymdirs` constants inside `fabriccli` rather than imported from `hardenshed.SandboxRoot`, which would pull the campaign engine's dependencies into the fabric command; `pairteardown` already joins the same constants.

## Tests

`TestSandboxedRoleGuard` gains a tier-0 subtest that runs each mutating form from four directories: the strand's own sandbox hub, passing; and another worktree's sandbox hub, a non-worktree directory's sandbox root, and the strand's own worktree outside its sandbox root, each refused.
Two rows pin `clone --into`: into the own sandbox root from the worktree passes, and out of the sandbox into another directory is refused.
The worktrees are fake `.git` directories written without git, and every passing verb stops at its own worktree resolution, so the subtest spawns nothing.
On the merge base every passing row fails, since the old guard refused them all.

## Merge danger

**Door:** two-way
**Blast radius:** fabric
