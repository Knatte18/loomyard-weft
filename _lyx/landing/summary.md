# No cgo in the module, so go vet never opens another worktree's files

`lyx gate test --tier 1` failed in one worktree with `can't parse raw cgo file: open <other worktree>/internal/reedengine/proctree_linux.go: no such file or directory` after that other worktree was torn down (issue #868).

## Cause

Under `-trimpath`, cmd/go keys a package's build action independently of the checkout directory, so every worktree of the hub shares one cache entry.
For a cgo package that entry includes the cgo-generated Go files, and their `//line` directives name the source by the absolute path of the checkout that first built it: cmd/go gives cgo no `-trimpath` rewrite outside overlays.
On a cache hit `go vet` reloads those files, and its `cgocall` analyzer opens the raw source each directive names.
So a tier1 `go vet` read `proctree_linux.go` from whichever worktree had seeded the entry, and failed once that worktree was gone.
A scratch module built in one directory and vetted in a copy, with a scratch `GOCACHE`, reproduces it.
A cgo dependency in the module cache is unaffected, since its path is the same in every worktree.

## Change

- `internal/reedengine`: `paneProcessAge` took the unit of a `/proc/<pid>/stat` start time from `sysconf(_SC_CLK_TCK)`, the module's only cgo use.
  It now reads the auxiliary vector's `AT_CLKTCK` entry through `golang.org/x/sys/unix.Auxv`, an existing direct dependency, and falls back to 100 when the entry is absent or the vector unreadable, as glibc's `sysconf` does.
  The pure `clockTicksFromAuxv` lives in `proctree.go` with a table test; the existing `PlatformProbe` subtest still checks a real age on Linux.
- `cmd/lyx/nocgo_test.go`: a `tier0` `//lyx:guard` scan fails for every Go file of the module that imports `"C"`, naming the way forward of reaching the C code through a module-cache dependency, with fixture rows for a plain, grouped and test-file import and for mentions in a comment or string.
- `pattern/PATTERN-gate-slot.md` and `internal/gateslot`'s doc state the rule beside the `-trimpath` bullet; the `PATTERN.md` overview line is unchanged.

The tree-sitter grammars still link through cgo from the module cache, so a build still needs `CGO_ENABLED=1`.

## Design decisions

- The fix removes the trigger rather than working around the cache: it keeps the shared cache `-trimpath` exists for and keeps `go vet`'s `cgocall` analyzer on, and it needs no change to the hub's `gate.yaml`.
- A per-worktree `GOCACHE` was not taken, since every fresh worktree would build cold.
- Detect-and-retry in `gatecli` was not taken: `go clean -cache` under other held slots breaks their builds, a `-a` retry rebuilds everything and only moves the poisoned path to the retrying worktree, and the error comes back after each teardown.
- `go vet -cgocall=false` would also avoid it, but the tier commands are the hub's, and `GOFLAGS` cannot carry the flag because `go build` and `go list` refuse it.
  Another Go hub with cgo in its own module can still hit the failure; that hub's way out is `-cgocall=false` on its tier's `go vet`.

## Verification

`lyx gate test --wait --gate darn-write` over the diff passed tier0, tier1, with its `go build` and `go vet` steps under every tag, and tier2.
