# Worktree-Teardown archive tag push races the weft branch push and fails with missing necessary objects

`archive-tag-push-race` · Note · bug, batten, fabric

## Seen (2026-11-02, another hub, darn run)

Worktree-Teardown failed: the push of the archive tag `archive/<slug>/<sha>` to the hub's weft remote was rejected with "remote rejected ... (missing necessary objects)".
A few minutes later `git ls-remote` showed the run's weft branch at that same commit on the remote, so the commit did reach it, apparently after the tag push ran.
The tackler then handed the stop back on a false premise: it read the commit as absent from the weft remote and asked the parent which remote to archive to.

## Workaround

A hand pass, `lym batten run <slug>`, redid the teardown and ended done.

## Expected

- The archive step waits for, or performs, the weft branch push before it pushes the archive tag, or retries the tag push on "missing necessary objects" once the branch is on the remote.
- The tackler checks `git ls-remote` before concluding that a commit is missing from a remote.
