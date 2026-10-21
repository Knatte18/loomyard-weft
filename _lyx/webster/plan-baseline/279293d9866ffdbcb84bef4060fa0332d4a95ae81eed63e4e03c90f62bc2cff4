# Card 3 — gate-slot-drain

**Create:**
- `plan:internal/gateslot#Pool.AcquireSlots` -> `func (p *Pool) AcquireSlots(ctx context.Context, h Holder, count int) (*Lease, error)`
- `plan:internal/gateslot#Lease.InheritEnv` -> `func (l *Lease) InheritEnv(base []string) []string`
- `plan:internal/gateslot#TestAcquireSlots` -> `func TestAcquireSlots(t *testing.T)`

**Edit:**
- `internal/gateslot#Holder`
- `internal/gateslot#Lease`
- `internal/gateslot#Pool.Acquire`
- `internal/gateslot#Pool.tryAcquireAny`
- `internal/gateslot#Lease.Release`
- `internal/gateslot#TestLeaseEnv`
- `internal/gateslot/doc.go`

**Uses:**
- `internal/gateslot#newPool`
- `internal/gateslot#shortContext`

**Intent:**
Give a tier the slots it asks for, with a drain that keeps a multi-slot tier from starving (D6).

`AcquireSlots(ctx, h, count)` with `count` 1 behaves as `Acquire`.
With `count` above 1, or 0 meaning every slot (`slots: all`), it clamps the count to the current `Limits().Slots`, takes the pool's drain lock at `<Dir>/drain.lock` (an OS file lock, so process death releases it), then takes slots one at a time in ascending slot order, holding each it has while it waits for the next, and releases the drain lock once it holds all of them.
Ascending acquisition means two multi-slot waiters never deadlock: the drain admits one at a time, and every single-slot holder finishes.
A tier holder finishes within its own tier timeout and a card `**Verify:**` holder within the runner's timeout; the plan glyph type load is bounded by its load, and `cmd/testtiming`, which the operator runs by hand, only by its own run, so a drain waiter queued behind a timing run waits for it.
While another process holds the drain lock, `Acquire` waits, polling as it does for a held slot, so new single-slot acquires queue behind the drain; the drain holder's own slot takes go through the internal path that skips that check.
A cancelled context releases every slot and the drain lock already taken and returns its error with nothing held.

`Holder` gains `Tier string` (`tier`, the tier key such as `tier0`, omitted when empty for a holder that is no tier run), so the holder record names the tier for the future `lyx gate status`.
`Lease` holds every lock it took; `Release` releases them all and removes their holder records.
`InheritEnv(base)` returns `base` with only `InheritEnv` set, naming the lease's first slot's lock path, so a nested gate run inside a multi-slot tier inherits it; it appends no GOFLAGS, since a tier's command and `env` set its parallelism.
`Env` keeps its GOFLAGS behaviour for the non-tier holders until card 19 moves them.
`Inherited` accepts any held slot lock of the pool, as today, so an inherited run acquires nothing whatever its tier asks for.

`doc.go`'s Slots and Inheritance sections describe the drain, the ordered acquisition and `InheritEnv`.

Tests: the new `TestAcquireSlots` covers, over a temp pool with injected short polls and in-process holders only, so it spawns nothing: a count of 1 taking one slot as `Acquire` does; a two-slot acquire on a free three-slot pool holding slots 1 and 2, its holder records naming the tier; `0` and a count above the pool's slots each clamped to every slot; a multi-slot waiter holding slot 1 while slot 2 stays held, then completing once it frees; a single-slot `Acquire` blocked while an in-process multi-slot waiter holds the drain lock and succeeding after that waiter completes and releases; two concurrent multi-slot acquires both completing; and a cancelled multi-slot wait releasing what it held and the drain lock.
`TestLeaseEnv` gains the row that `InheritEnv` sets only the inheritance variable.

**ImpactSummary:** the single-slot `Acquire` keeps its signature and gains only the drain wait; every current caller is unchanged, and the holder record gains one omitted-when-zero field.
