# Card 12 — gate-entry-exemption

**Create:**
- `internal/pauserecord#SetGateExempt`

**Edit:**
- `internal/pauserecord#Gate`
- `internal/pauserecord#Waiter`
- `internal/pauserecord#WaitUncovered`
- `internal/pauserecord#Network`
- `internal/pauserecord/doc.go`
- `internal/pauserecord#TestNetwork`

**Intent:** D8's wake exemption: the hibernate daemon's reconcile passes the network gate while the hub pause entry its own request owns stands, and nothing else does.
`Gate` gains an exempt entry (scope and pause time, zero for none); `SetGateExempt` replaces the installed gate's exempt entry, as `SetGateSlug` replaces its slug, does nothing with no gate installed, and returns the function that restores the previous value.
`Waiter` gains `ExemptEntry func(Entry) bool`; `WaitUncovered` passes when every live entry covering the slug is one `ExemptEntry` accepts, and holds as today while any other covering entry stands.
`Network` hands the gate's exempt entry to its waiter, matching by scope and `PausedAt`, so only the one entry passes, and only calls of the process that set it.
Bound: an exemption covers only the process that sets it and only the named entry; any other entry, a slug entry or a newer hub entry, still holds the call, and the daemon restores the gate when its reconcile ends (card 23).
`doc.go`'s Network paragraph names the exemption.

`TestNetwork` gains rows: with the exempt entry the only live hub entry, a call passes at once; with the exempt entry and a second live entry covering the slug, the call holds until the second is lifted; after the restore, the exempt entry holds again; `SetGateExempt` with no gate installed is a no-op.

**ImpactSummary:** Two struct fields and one new setter in a leaf package; every existing gate and waiter carries a zero exemption, so behaviour is unchanged until card 23 sets one.

**Commit:** 12: pauserecord: a gate may pass one named pause entry
