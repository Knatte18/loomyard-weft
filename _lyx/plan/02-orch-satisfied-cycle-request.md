# Card 2 — orch-satisfied-cycle-request

**Edit:**
- `internal/orchengine/watcher.go`
- `internal/orchengine/watcher_compact_test.go`
- `internal/orchengine/watcher_test.go`
- `internal/orchengine/doc.go`

**Intent:**
D2: a same-mode cycle request queued during a cycle is satisfied by that cycle instead of starting a second one.

`tickIdle`'s entry into the handoff-requested phase already clears any request pending when the cycle starts, so every request found later was made after the cycle started; no cycle start time is stored.
At the moment the cycle's own action takes effect, a pending request (`CycleRequested`) of the same mode is removed with `ClearCycleRequest` and one `logger.Info` line naming its mode and `RequestedAt`:
- compact (`CycleCompact`): in `Watcher.tickCompacting`'s completion branch from card 1, when the request's `RequestedAt` is not after the boundary's `At`;
- clear (`CycleClear`): in `Watcher.tickClearing`, when the `/clear` is confirmed typed (`st.PhaseInjected` true at the point it moves on to the reload).

A request of the other mode, and a compact request made after the boundary, stay pending for `tickIdle` as today.
Bound, stated in the code comment: it removes only a request made after the cycle started and before its action took effect, for the same thing the action did; the request time (CLI-written) and the boundary time (transcript) are wall clock on one host.
A `CycleRequested` or `ClearCycleRequest` error is returned like the other state errors in these methods.
Add the rule to the cycle-request section of `internal/orchengine/doc.go`.

Tests, as rows or cases beside the existing cycle-request tests:
- compact: a compact request made during the handoff phase, and one made during compacting before the boundary, are removed when the compaction completes; one made after the boundary, and a clear request made during the cycle, stay pending (`watcher_compact_test.go`);
- clear: a clear request made after a clear cycle started and before its `/clear` was confirmed is removed; a compact request made during it stays pending (`watcher_test.go`).

**ImpactSummary:** two watcher methods in `orchengine`; the request marker's on-disk format and the CLI that writes it are unchanged.
