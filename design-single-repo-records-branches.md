# One-repository hubs redesigned: records on their own branches, the layout known only to fabric

`single-repo-records-branches` · Task · feature, fabric · draft

## Why (operator, 2026-11-19)

`single-repo-fabric-mode` (f06acf39) put `_lym` records as tracked files on each code branch.
Records commits then sit in the code history, so every module that reads git sees them,
and `single-repo-code-identity` had to strip records paths in about 15 packages outside fabric.
That breaks the hub's rule: only fabric knows the layout.
Whether `_lym` and `.lym` live in a sibling weft repository or elsewhere is invisible to every module but fabric.
`single-repo-code-identity` was dropped for this reason.

## Direction

A one-repository hub keeps records off the code branches, the way the board already sits on the orphan branch `lym-bolt` checked out as `_bolt`:
records live on their own branches of the same repository, each checked out as a worktree that `_lym` joins as weft does today.
Seen from outside it is one repository and one remote; the code history holds code only, so no module outside fabric changes.

## To decide in Discussion

- The records branch scheme: one per task, one shared, or another, and how merges, landings, replay and teardown treat them.
- What of f06acf39 stays (clone, `lym-bolt` and `_bolt`, the guarded prime push, merge and replay parts) and what is removed,
  including its `lymcwd.OneRepository` branches in `battencli/commitstatus.go` and `loomcli/wiring.go`.
- The layout probe that since f06acf39 warns on every board read about entries with no worktree on disk.
