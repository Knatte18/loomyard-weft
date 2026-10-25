# Card 11 — idlewatch-snapshot

**Create:**
- `internal/parentlock#IdleWatch.Waiting`
- `internal/parentlock#RestoreIdleWatch`
- `internal/parentlock#TestIdleWatch_RestoreKeepsTheWait`

**Intent:**
Let the idle rule's watch be saved and restored, so batten's short passes keep the parent lock's idle bound across processes (D4's `lockWatch` inventory item).
`Waiting` returns a copy of the first-seen-waiting readings by agent name, on the caller's awake clock; `RestoreIdleWatch` returns a watch with `Bound` and those readings, which its next `Observe` treats as the earlier sightings.
The readings are meaningful only against the same awake clock, which the caller keeps across processes.
`TestIdleWatch_RestoreKeepsTheWait` (tier0, parallel): a watch that saw agent `a` waiting at awake 10s, saved and restored, reports idle at awake 10s plus the bound with `a` still waiting and no busy agent, and a restore with no readings reports not idle at that point.
