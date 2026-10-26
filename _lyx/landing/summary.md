# Drop the module's cgo use so gate vet never reads another worktree

`lyx gate test --tier 1` could fail before any test with `can't parse raw cgo file: open <other worktree>/internal/reedengine/proctree_linux.go: no such file or directory` once the worktree that first built `internal/reedengine` was torn down.
Under `-trimpath` the shared Go build cache keys a build independently of the checkout directory, but the cgo-generated files it keeps carry `//line` directives with the absolute path of the checkout that built them.
A later `go vet` in another worktree reloads those files, and its `cgocall` analyzer opens the raw source each directive names.
Only a cgo package in a checkout-dependent directory triggers this; a cgo package in the module cache sits at the same path in every worktree.

Closes #868

## Changes

- `internal/reedengine`: `paneProcessAge` no longer calls `sysconf(_SC_CLK_TCK)` through cgo.
  It reads the clock tick rate from the ELF auxiliary vector's `AT_CLKTCK` entry through `golang.org/x/sys/unix.Auxv`, via the new `clockTicksFromAuxv`, and falls back to 100 when the entry is absent or not positive, as glibc does.
  With that call gone, no package of the module uses cgo.
- `cmd/lyx/nocgo_test.go`: a `tier0` guard fails for every Go file of the module that imports `"C"`, naming the file and the way forward.
  It scans the module's own files only, so cgo dependencies such as the tree-sitter grammars stay allowed.
- `pattern/PATTERN-gate-slot.md` and the `internal/gateslot` package doc state the no-cgo rule beside the `-trimpath` rule.

## Design decisions

- Removing the trigger was chosen over per-worktree Go caches or detect-and-retry in `gatecli`.
  It keeps the shared `-trimpath` build cache and keeps `go vet`'s `cgocall` analyzer on, and it fixes every tier and every worktree.
- The hub's `gate.yaml` tier commands are untouched.
- `AT_CLKTCK` is taken as 17 on every Linux architecture, per the kernel's `auxvec.h`.
- `golang.org/x/sys/unix` is already a direct dependency, and `internal/reedengine` is no leaf package, so the import is allowed.

## To check by hand

- Cache entries already poisoned for `internal/reedengine` stop mattering once the package has no cgo, since its action ID changes.
  A worktree still on an older commit keeps the old ID until it merges, so it can still hit the stale entry; `go clean -cache` while no gate slot is held clears it.
- `paneProcessAge` returning the same age as before was covered by unit tests of `clockTicksFromAuxv` and the host-vector test, not by a run against a live tmux pane.

## Tests

- `clockTicksFromAuxv` is tested for a declared rate, a missing entry, a zero rate and a nil vector, and a Linux test checks that the host's real vector yields its `AT_CLKTCK` value.
- The guard is tested on fixture trees: it fails on a plain, grouped and test-file `import "C"`, and passes on a mention in a comment or string.
- The guard also runs over the real tree.
