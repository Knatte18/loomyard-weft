# Codex sessions can commit in a worktree and use the Go cache

A Codex writer in a linked worktree could not commit.
The launch line runs `-s workspace-write` and added only the records checkout as a writable root.
A linked worktree's git dir lies under the main checkout's `.git/worktrees/<slug>`, outside every writable root, so `git commit` failed on `index.lock` with `Read-only file system`.
The failure is deterministic, so no darn or tapestry run with a Codex writer could land.
The default Go build cache under `~/.cache/go-build` was read-only for the same reason, so the first `go test` failed.

## What changes

`codexengine` now adds these writable roots to the launch line, in this order, each symlink-resolved:

- The records checkout, as before.
- The checkout's git dir and the common git dir it links to.
  Both are read from the `.git` entries, as the trusted roots are, without running git.
  For a plain clone the two are the same folder and appear once.
- The folders named by the new `codex.yaml` key `writable_roots` that exist.
  The template sets `[~/.cache/go-build]`; a leading `~/` is the user's home directory.

A `writable_roots` folder that does not exist is left out, so a toolchain the machine lacks adds nothing.
An empty or relative value is refused by its key before anything is written.

## Design decisions

- `writable_roots` is `configengine.CategoryGuard`, so a local override may not set it.
  A writable root widens the sandbox, and a per-worktree override should not do that.
  `internal/configreg`'s classification test lists the key beside `binary`.
- The git dirs are always added, not configured.
  Every Codex writer in a pair worktree needs them to commit, so they are not an operator choice.
- The cache is a configured root rather than a `GOCACHE` override.
  The session's `go` then keeps using the operator's existing cache.

## Docs

The `codexengine` package doc and `docs/overview.md` describe the added roots.

## To check by hand

- Run a darn task with a Codex writer in a linked worktree, as in the sandbox hub `lym-test-LYMHUB`, and confirm the writer commits and the run lands.
  This was not run; the tests cover the roots `Prepare` builds, not a live Codex sandbox.
- Confirm a first `go test` in that session runs without a `GOCACHE` override.
