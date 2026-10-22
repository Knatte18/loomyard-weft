# Teardown's archive-tag push has no retry for a transient remote rejection

`retry-archive-tag-push` · Note · bug, batten, fabric

Imported from [issue #804](https://github.com/Knatte18/loomyard/issues/804).

## Seen (orch, 2026-10-21, OSL-1046)

`origin-reconcile`'s Worktree-Teardown pushes the archive tag `archive/<slug>/<sha>` to the weft remote before removing the worktree.
GitHub rejected that push with a bare `(failed)` twice, and each time the row failed and the run halted.
The third `lyx batten run origin-reconcile --window` pushed the tag and finished the teardown, so the failure was transient.

## Workaround

Re-run `lyx batten run <slug> --window` until the push goes through.

## Expected

The archive-tag push retries a transient remote rejection a few times with backoff before failing the row, as a transient network error should not need an orch turn.
