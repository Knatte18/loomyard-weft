# Gate: run go test with -count=1 so the cache stops filling with logs

Gate runs of `go test` filled `~/.cache/go-build` with hundreds of GB of test logs a day; the cache on neve reached about 750 GB.
With test caching on, Go logs every environment variable a test reads and every file it opens or stats, to judge whether a cached result still holds.
lym's tests build whole hubs with git repositories under fresh random temp directories, so one log reaches about 700 MB and no cached result is ever reused.
The gate already carries a green record over an unchanged tree, so the test result cache adds nothing to gate runs.

Every tier command and every non-tier slot holder's child now gets `-count=1` first in its GOFLAGS.
`go test` then neither reads nor writes its test result cache, so it writes no test log.
The build cache is untouched, and `-trimpath` still lets a fresh worktree reuse it.
No hub config changes and no reconcile of existing tier commands is needed.

## Design decisions

- `gateslot.TestCacheOffFlag` holds the flag, and `gateslot.WithTestCacheOff` puts it ahead of an environment's last GOFLAGS entry.
  The result holds one GOFLAGS entry, and the input slice is left as it was.
  The GOFLAGS key match is case-insensitive, as in `HolderEnv`.
- `runTier` in `internal/gatecli` applies `WithTestCacheOff` to the environment of every tier invocation.
- `gateslot.HolderEnv` now orders GOFLAGS as `-count=1`, the base's value, `-p=<n>`, `-trimpath`.
  `internal/planglyph`'s `loadEnv` takes this through `HolderEnv`, so its `go list` loads carry the flag too.
- The flag comes first on purpose: a hub's own `-count` in GOFLAGS, or one in the words after `--`, still wins.
- The `gatecli` and `gateslot` docs now say a fresh worktree reuses the build cache, not "the build and test cache".
  `gatecli` gains a "Test result cache" section.

## To check by hand

- Confirm with one real run that `-count=1` writes no test log: after a `lym gate test` run, no new large `*-d` file appears under `~/.cache/go-build`.
  That run was not made.
