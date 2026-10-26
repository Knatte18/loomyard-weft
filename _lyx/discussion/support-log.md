# Support log — gate-stale-vet-cache-path

## Interview

A darn run holds no interview; the writer read the board entry, the hub's `gate.yaml`, `internal/gateslot`'s doc, `pattern/PATTERN-gate-slot.md` and cmd/go's `loadCachedVet`, and reproduced the failure in a scratch module with a scratch `GOCACHE`.

## Rejected alternatives

- A per-worktree `GOCACHE`: every fresh worktree would rebuild from cold, which defeats the shared cache `-trimpath` exists for under `PATTERN-gate-slot`.
- Detect the error in `gatecli` and retry with `go clean -cache`: cleaning the shared cache under other held slots breaks their in-flight builds, and the error recurs after every teardown of the worktree that seeded the entry.
- Detect and retry with `GOFLAGS=-a`: rebuilds the standard library and every dependency on each hit, and only moves the poisoned path to the retrying worktree.
- `go vet -cgocall=false` in the tier commands: the commands are the hub's, on Bolt, and `GOFLAGS` cannot carry the flag, since `go build` and `go list` refuse it as unknown; it also drops the `cgocall` check.
- A per-worktree `CGO_CFLAGS` to split the cgo action IDs: it also splits every cgo dependency's ID, so the tree-sitter grammars would recompile in every fresh worktree.
- Hard-coding 100 ticks per second: correct on the mainstream architectures but not on all, while the auxiliary vector is exact.

## Review rounds

_No rounds yet._

## Question ledger

- Fix in lyx code or in the hub's tier commands? Self-pick: lyx code, by removing the module's only cgo use, since the tier commands are the operator's and the code fix needs no config change. Resolved.
- Add a guard? Self-pick: yes, a `tier0` scan, since a later cgo file in the module would bring the failure back silently. Resolved.
