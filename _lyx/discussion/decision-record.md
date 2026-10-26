# Decision record — gate-stale-vet-cache-path

## Goal

A gate run never reads a file from another worktree, so `lyx gate test --tier 1` stops failing with `can't parse raw cgo file: open <other worktree>/internal/reedengine/proctree_linux.go: no such file or directory` after a worktree is torn down.

## Scope

- In: removing the main module's only cgo use, the `sysconf(_SC_CLK_TCK)` call in `internal/reedengine/proctree_linux.go`, and a guard that keeps cgo out of the module's own packages.
- In: the docs that state the invariant, `pattern/PATTERN-gate-slot.md` and the gate's package doc.
- Out: changing the hub's `gate.yaml` tier commands, which live on Bolt and are the operator's.
- Out: per-worktree Go caches and detect-and-retry in `gatecli`.

## Decisions

### Root cause: cmd/go caches a cgo package's generated files with absolute source paths

Reproduced with a scratch module and a scratch `GOCACHE`: a `-trimpath` `go build` in directory A, then A removed, then `go vet` of an identical copy in directory B fails with `can't parse raw cgo file: open A/m.go`.
cmd/go keys the build action independently of the directory under `-trimpath`, but the cgo-generated files it caches carry `//line` directives with A's absolute path, since cgo gets no `-trimpath` rewrite outside overlays.
On a cache hit `go vet` reloads those files, and the `cgocall` analyzer opens the raw source each directive names.
Rationale: the failure is a toolchain behavior that only a cgo package in a checkout-dependent directory triggers; a cgo package in the module cache sits at the same path in every worktree.

### Fix: the module's own packages import no `"C"`

`paneProcessAge` reads the clock tick rate from the ELF auxiliary vector's `AT_CLKTCK` entry through `golang.org/x/sys/unix.Auxv`, falling back to 100 when the entry is absent, as glibc's `sysconf(_SC_CLK_TCK)` does.
With that call gone no package of the module uses cgo, so no cached cgo output names a worktree path.
Rationale: it removes the trigger for every tier and every worktree, keeps the shared build cache `-trimpath` exists for, and keeps `go vet`'s `cgocall` analyzer on.

### Guard: a `tier0` scan refuses `import "C"` in the module

A `//lyx:guard` scan in `cmd/lyx` fails for every Go file of the module that imports `"C"`, naming the file and the way forward of reaching the C code through a dependency in the module cache.
Bound: it scans the module's own files only, so cgo dependencies such as the tree-sitter grammars stay allowed.

## Constraints

- `PATTERN-gate-slot`: every Go gate build keeps `-trimpath` and the shared cache.
- `PATTERN-test-speed`: the guard is a `tier0` test, offline and parallel.
- `PATTERN-leaf-packages`: `internal/reedengine` is no leaf package, so importing `golang.org/x/sys/unix`, already a direct dependency, is allowed.

## Auto-mode assumptions

- The hub's tier commands stay as they are; the fix needs no `gate.yaml` change.
- `AT_CLKTCK` is 17 on every Linux architecture, per the kernel's `include/uapi/linux/auxvec.h`.

## Open risks

- Cache entries already poisoned for `internal/reedengine` stop mattering once the package has no cgo, since its action ID changes; a worktree still on an older commit keeps the old ID until it merges.

## Acceptance criteria

- `internal/reedengine` builds and vets with no cgo file, and `paneProcessAge` returns the same age as before on Linux.
- The guard fails on a fixture file importing `"C"`, passes on a file that mentions `import "C"` only in a comment or string, and passes on the real tree.
- `lyx gate test` over the diff is green.

## Notes for the plan writer

- The `PATTERN-gate-slot` overview line stays as it is; the no-cgo rule goes into its background file beside the `-trimpath` bullet.
