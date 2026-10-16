# Card 8 — creel-interrupt-clear

**Create:**
- `plan:internal/creelengine#ErrClearFailed` -> `var ErrClearFailed error`
- `plan:internal/creelengine#Store.paneLock` -> `func (s *Store) paneLock(receiver string) *sync.Mutex`
- `plan:internal/testkit/creelfake#Delivery.SetInterruptFunc` -> `func (d *Delivery) SetInterruptFunc(fn func(strand string) error)`
- `plan:internal/creelcli#restoringEngine` -> `type restoringEngine struct`
- `plan:internal/creelcli#stepClock` -> `type stepClock struct`

**Edit:**
- `internal/creelengine#Delivery`
- `internal/creelengine#Store`
- `internal/creelengine#OpenStore`
- `internal/creelengine#Store.interruptTarget`
- `internal/creelengine#worker.probe`
- `internal/creelengine/doc.go`
- `internal/creelengine_test#TestStore_InjectAndHold`
- `internal/creelengine_test#TestScheduler_Injects`
- `internal/testkit/creelfake#Delivery`
- `internal/testkit/creelfake#Delivery.Interrupt`
- `internal/creelcli#shuttleOps.Interrupt`
- `internal/creelcli#mapDeliveryError`
- `internal/creelcli#TestHubDelivery`

**Uses:**
- `plan:internal/shuttleengine#Runner.InterruptRestoring`
- `plan:internal/shuttleengine#ErrInterruptClearFailed`
- `internal/creelengine_test#env`
- `internal/creelengine_test#newEnv`
- `internal/creelengine_test#env.start`
- `internal/creelengine_test#env.tick`
- `internal/creelengine_test#env.probeWaits`
- `internal/creelengine_test#env.typed`
- `internal/creelengine_test#eventsOf`
- `internal/creelcli#newStubbedDelivery`
- `internal/testkit/shuttlefake#Reed`
- `internal/testkit/shuttlefake#Engine`
- `internal/shuttleengine#NewRunner`
- `internal/shuttleengine#NewEngineSet`
- `internal/shuttleengine#Runner.SetClock`

**Intent:** Decision D3, creel half: `lyx creel interrupt` and `inject --interrupt` use shuttle's restoring interrupt, a clear that fails after the key still counts as a sent interrupt, and no delivery reaches the receiver while the interrupt runs.

The seam: `Delivery.Interrupt`'s doc adds that an error wrapping `ErrClearFailed` means the interrupt key was sent and only the clear after it failed.
`internal/creelcli`'s `shuttleOps.Interrupt` calls `Runner.InterruptRestoring` instead of `Runner.Interrupt`, so both verbs get the clear, and `mapDeliveryError` maps an error wrapping `shuttleengine.ErrInterruptClearFailed` onto one wrapping `creelengine.ErrClearFailed` that keeps shuttle's message.
`Runner.Interrupt` keeps its other callers.

The engine: `Store.interruptTarget` treats an `ErrClearFailed` answer as sent: it logs the clear failure at Warn, journals `interrupted` as for a plain success and returns true with no error, so `interrupt` reports `sent:true` and `inject --interrupt` still queues its line at the head.
The suspension: `Store.paneLock(receiver)` returns the one mutex of that receiver, from a map in `Store` created in `OpenStore`.
`Store.interruptTarget` holds it around `Delivery.Interrupt` only, after the pause and hold checks, and wakes the receiver once it releases it; `worker.probe` takes it before it asks `Delivery.Idle` and holds it through the send or reminder, so while creel's interrupt runs, its settles and its clear included, the receiver's worker neither probes nor sends nor notes a stall, and its first probe after the interrupt is one fresh reading.
Bound: the lock serialises one receiver's interrupt with its own deliveries only, never another receiver's, and an interrupt waits at most one in-flight delivery of that receiver, which a busy send's own window already bounds.

Tests:
`TestStore_InjectAndHold`'s `Inject_InterruptQueuesAtTheHeadAndFailsOnOtherErrors` row gains, beside its `boom` case, a scripted interrupt error wrapping `ErrClearFailed`: `Interrupt` returns true and no error, the log holds `interrupted`, and an `Inject` with interrupt set still queues its line at the head.
`TestScheduler_Injects` gains a row where `SetInterruptFunc` blocks an interrupt on a channel while an inject and a reminder are pending for the receiver: the row moves the clock with `e.clock.Advance` past several probe waits, never `e.tick`, since a worker held on the receiver's lock arms no new wait, and asserts `Idle` was asked nothing more about the receiver and nothing was typed; it then closes the channel, so the interrupt returns before the scheduler's cleanup stops it, and the queued inject is typed.
A second row pins the wake: a receiver read not idle backs its worker off into a long wait, the row then reads it idle and runs an interrupt that returns at once, and the queued inject is typed with no further clock move, so the interrupt's end woke the worker.
The fake: `creelfake.Delivery` gains an interrupt function its `Interrupt` calls, without the fake's lock held, after recording the call, in place of the scripted error when set; the kit's package doc names it.
`TestHubDelivery`'s sentinel-mapping row checks, for `Interrupt` only, that a stub error wrapping `shuttleengine.ErrInterruptClearFailed` comes back wrapping `creelengine.ErrClearFailed`.
`TestHubDelivery` gains a row that pins the wiring, building a `shuttleOps` over a `shuttleengine.Runner` on a `shuttlefake.Reed` with a live strand and no run record, and `restoringEngine`, a `shuttlefake.Engine` with the box reader and the restored-prompt clear whose pane shows a running turn over an empty box before the `Escape` and a restored line after it, on a `stepClock` the runner's settles advance: `shuttleOps.Interrupt` sends the `Escape` and then a `C-u`, which only `Runner.InterruptRestoring` sends, on a strand with no run record as a hand-added one has.

`internal/creelengine/doc.go`'s Inject and interrupt paragraph says the interrupt clears a prompt the provider restored, a clear failure still counts as sent, and no delivery reaches the receiver while the interrupt runs.

**ImpactSummary:** Swaps creelcli's interrupt to the restoring entry point and adds a per-receiver lock between the interrupt and the worker; no other caller of Runner.Interrupt changes.
