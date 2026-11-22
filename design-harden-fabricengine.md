# Hardener campaign on fabric, one-repository layout

`harden-fabricengine` · Task · bug, fabric · running · PR-Review

## Focus

Harden only the one-repository hub layout that 57015a52 added: one origin holds the code under `refs/heads/` and `refs/tags/`, and the records and board under `refs/lym/heads/` and `refs/lym/tags/`.
Everything else in fabric, the two-repository layout included, is out of scope, and no round is steered outside this focus.

Drive each of these live on a one-repository sandbox hub (`lym harden sandbox new --single-repo`), with real git and real `lym` verbs, and improvise scenarios beyond them:

- **Seeding**: `lym fabric clone --single-repo <url>` on an empty origin and on one with code only.
  The board ref is pushed last; kill or fail the push at each step and check that nothing on origin reads as a hub, the hub directory is gone and the records branch is restored.
- **Joining**: a second `lym fabric clone <url>` of a seeded origin takes the one-repository layout, gets the refspecs and `branch.main.merge refs/lym/heads/main`, and sees the board.
- **Refusals**: `--single-repo` on an origin that already holds a namespaced board, a records repository on the default branch, a second URL, or the old layout's `lym-bolt` branch or root `.lym-anchor`/`_lym`.
  Each refuses before creating anything and names its way forward.
- **Namespace leaks**: no records ref ever lands under plain `refs/heads/` or `refs/tags/` on origin, and the code store never fetches or pushes a records ref.
  Check every path: add, push, pull, replay, hand-over, teardown, archive, remote cleanup.
  The archive tag of a discarded task branch was seen in plain `refs/tags/` in a sandbox trial; confirm or refute it.
- **Two clones at once**: pushes and pulls from two clones of the same hub, a board write from each, a lease that loses.
- **Remote delete**: deleting a missing namespaced branch records no deletion; deleting a present one removes it under `refs/lym/heads/`.
- **Prime records branch**: `lym.primeRecordsBranch` is written once by the clone; a store without it fails closed with a way forward, and the orphan sweep keeps the branch.
- **Layout detection**: the records store's fetch refspec decides the layout; an unreadable remote config is an error, never read as two repositories.
- **Config read**: git's `@{u}` works on a namespaced branch, and `gitrepo`'s tolerant config read accepts `refs/lym/heads/<b>`.

Every finding states the repro steps that showed it.
