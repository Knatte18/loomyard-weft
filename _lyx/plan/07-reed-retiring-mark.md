# Card 7 — reed-retiring-mark

**Edit:**
- `internal/reedengine#Strand`
- `internal/reedengine#StrandStatus`
- `internal/reedengine#Engine.Status`
- `internal/reedengine#Engine.RemoveStrand`
- `internal/reedcli#reedCLI.removeCmd`
- `internal/reedcli#reedCLI.statusCmd`

**Create:**
- `internal/reedengine#Engine.MarkRetiring`
- `internal/reedengine/retiring_test.go`
- `internal/reedcli/removedetach_test.go`

**Intent:**
Implements reed's half of D7.

reedengine:
- `Strand` gains `Retiring bool` (json `retiring`, omitempty) and `StrandStatus` gains `Retiring bool`; `Status` copies it across.
- A new method `Engine.MarkRetiring(guid string, retiring bool) error` sets or clears the flag under the op lock and persists state.
  It needs no live session, since it touches only the strand table.
- A new exported sentinel `ErrUnknownStrand` is wrapped by `removeStrandLocked`'s unknown-guid error (and by `MarkRetiring`'s), so a caller can tell "already gone" from a failure.
- The `state.go` header comment's dumb-carrier paragraph names `Retiring` as reed's own lifecycle bookkeeping, set by a reed verb rather than written by a caller.

reedcli `remove`:
- With `--name --detach`, after resolving the guid and before spawning the detached remover, it calls `MarkRetiring(guid, true)`.
  When `spawnDetachedRemove` fails it calls `MarkRetiring(guid, false)` before reporting the error, so no strand is left marked with no remover behind it.
  A failed clear is logged at Warn and named in the error.
- The detached remover (the `--wait-pid` path) treats `errors.Is(err, ErrUnknownStrand)` as a clean no-op: an ok envelope with an empty `removed` list.
  This makes the late remover harmless after `lyx loom start` has already replaced the strand under a new guid.
- The `remove` help text says `--detach` marks the strand retiring first and that the detached remover is a no-op for a guid already gone.

reedcli `status`:
- Each strand in the `lyx reed status` envelope gains a `retiring` key, so the operator sees the mark D7 says reed's strand status exposes.

Bound: the mark records only that a caller of `reed remove --detach` requested the strand's removal.
It grants nothing by itself: `lyx loom start` (card 8) and the teardown quiet check (card 9) only complete that request, and a strand without the mark keeps every existing handling.
The no-op applies only to the detached remover's unknown-guid case; a positional `remove <guid>` typed by an operator still errors on an unknown guid.

Tests:
- `retiring_test.go` (untagged) covers `MarkRetiring` round-tripping through saved state and `Status`, and `MarkRetiring` on an unknown guid wrapping `ErrUnknownStrand`.
  It also covers `removeStrandLocked`'s unknown-guid error wrapping `ErrUnknownStrand`.
- `removedetach_test.go` (untagged) covers the mark-then-spawn order and the clear on a failed spawn through a fake engine seam (adding the narrow seam on `reedCLI` this needs).
  It also covers the `--wait-pid` path on an unknown guid answering ok with an empty list.
  Through the same seam, it covers `status` reporting `retiring: true` for a marked strand and `false` for an unmarked one.

**ImpactSummary:** Adds an optional persisted flag and one engine method; reed remove --detach gains a pre-spawn write and its detached child tolerates an already-removed guid; reed status gains one strand key.

**Commit:** `7: reed marks a self-removing strand as retiring`
