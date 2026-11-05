# A Codex session can write the checkout's git dirs and a build cache

A Codex writer in a pair worktree could not commit.
The launch line ran `-s workspace-write` with the records checkout as its one extra writable root.
A linked worktree's git dir lies under the main checkout's `.git/worktrees/<slug>`, outside the workspace and every writable root, so the sandbox refused `index.lock` and no darn or tapestry run with a Codex writer could land.
The same sandbox left the Go build cache under `~/.cache/go-build` read-only, so a session's first `go test` failed.

## What changes

- `codexengine` reads the checkout's git dir and common git dir from the `.git` entries, as it already reads the trust roots, and adds both as writable roots after the records checkout.
  A plain clone adds its one `.git` directory.
  Both paths are symlink-resolved, since Codex keys a path by its resolved target.
- `codex.yaml` gains `writable_roots`, a list of extra folders every Codex session may write; the template names `~/.cache/go-build`.
  A leading `~/` is the user's home, an empty or relative value refuses the launch before anything is written, and a folder that does not exist is left out, so a machine without the cache adds nothing.
- `writable_roots` is a guard-category key: a local override may not set it, since a writable root widens the sandbox.
- The package doc, the `Prepare` and `readyHome` docs and the shuttle row of `docs/overview.md` describe the new roots.

## Design decisions

- The build cache is a config key, not a `GOCACHE` under the lym temp root as the entry suggested.
  A shared cache keeps a session's builds and its `lym gate test` runs warm, where a per-run temp cache would rebuild the standard library each session, and a config list keeps the engine free of a hard-coded language.
- The git dirs are always added, outside a hub too, since any linked worktree needs them to commit.

## Tests

- `TestPrepare_Launch` gains a plain-clone row and a configured-roots row (an existing root kept, a missing one dropped), and every row now expects the git dirs.
- `TestPrepare_RefusesBeforeWriting` gains a relative `writable_roots` row that refuses with the key named and leaves the lym home unmade.
- `TestOpenLaunch`, `TestPrepare_InstallsGuardsFromItsInputs` and `TestLoadConfig_Template` expect the new roots and template key.
- The fixture writes an empty `writable_roots` worktree layer, so no test reaches the real home's cache.

Not verified live: no Codex session ran against the new launch line in this change.
