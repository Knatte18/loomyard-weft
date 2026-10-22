# Teardown's archive-tag push has no retry for a transient remote rejection

`retry-archive-tag-push` · Task · bug, batten, fabric · running · Publish

Imported from [issue #804](https://github.com/Knatte18/loomyard/issues/804).

## Seen (orch, 2026-10-21, OSL-1046)

`origin-reconcile`'s Worktree-Teardown pushes the archive tag `archive/<slug>/<sha>` to the weft remote before removing the worktree.
GitHub rejected that push with a bare `(failed)` twice, and each time the row failed and the run halted.
The third `lyx batten run origin-reconcile --window` pushed the tag and finished the teardown, so the failure was transient.

## Workaround

Re-run `lyx batten run <slug> --window` until the push goes through.

## Expected

The archive-tag push retries a transient remote rejection a few times with backoff before failing the row, as a transient network error should not need an orch turn.

## Already in place (triage, 2026-10-22)

`archiveWeftTip` (`internal/fabricengine/archive.go`) already pushes through `gitrepo.PushWithRetry`, which retries a bare `(failed)` 4 times with 1 s, 2 s and 4 s backoff, and the failures happened despite it.
Operator decision (2026-10-22): the retry gets a longer backoff, and covers the rejection class actually seen if it is not the one `PushWithRetry` retries.
