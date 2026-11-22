# Let hardener rounds run fabric verbs in their own sandbox hubs

The task-session guard in `internal/fabriccli/roleguard.go` refused every mutating `lym fabric` verb by strand name alone.
A hardener round that built a sandbox hub with `lym harden sandbox new` was refused add, push, pull, sync, reconcile, checkout, remove, clone and `cleanup --apply` inside that hub.
Seven of twelve charter items in `harden-fabricengine` round 1 ended as could not verify.

The guard now also judges the directory the verb acts in.
A sandboxed role passes every verb when that directory lies at or under `.lym/harden-sandbox` of the role's own worktree.
The same verb aimed at the real hub, or at another worktree's sandbox root, stays refused.
The refusal text now names the sandbox root.

## Design decisions

- **Target directory.**
  It is the cwd carried on the command context, and for `clone` the destination `--into` names.
  `clone.go` now computes that destination in `cloneDestination`, so the guard and the verb read one definition.
- **Own worktree.**
  The strand name's slug and shortname must match the worktree that holds the sandbox root and the hub whose recorded shortname it is.
- **Links resolve first.**
  The target is resolved through the symlinks of its deepest existing ancestor, so a link counts where it points and a clone destination not yet made counts where it would be made.
  A dangling link on the way is refused.
- **Records sibling.**
  A worktree's `.lym` links into its records sibling, so a resolved path is mapped back to the worktree that owns that sibling before the checks above.
- **Limit.**
  The guard still stops a mistaken call and not a session that rewrites the strand variable or builds a sandbox root of its own; the comment on `sandboxedRoleRefusal` says so.
- `doc.go`, `fabric.go` and the harden review stencil `harden-rubric-review.md` state the exception.

## Check by hand

- The sandbox-root judgment through the records-sibling link (`inOwnSandboxRoot` and `worktreeOwningRecordsDir` in `roleguard.go`) was flagged as a blocking design finding and fixed in follow-up commits.
  Review that path mapping, and that a verb under another worktree's sandbox root is refused.
- A live run of a hardener round that drives the mutating verbs in a `lym harden sandbox new` hub was not done.

## Merge danger

**Door:** two-way
**Blast radius:** fabriccli
