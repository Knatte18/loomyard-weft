# Card 2 — reread-and-reset

**Edit:**
- `internal/orchengine/watcher.go`
- `internal/orchengine/watcher_test.go`
- `internal/orchengine/state.go`
- `internal/orchengine/state_test.go`
- `internal/orchengine/doc.go`

**Intent:**
Stop the watcher cycling on a stale reading (D2) and reset the reading on a fresh launch or adopt (D4).

In `Watcher.tickIdle`, once a hard or soft trigger has passed every gate (newest event a turn end, quiet period, deferral hold, idle probe), re-read the context through `w.newest` (always a turn end at that point), save the new reading to State, and fire only if the re-read reading still meets that trigger's threshold: the hard cap for a hard trigger, the soft threshold for a soft one.
A re-read below the hard cap but at or above the soft threshold does not upgrade a hard trigger into a soft firing on the same tick; the next tick re-evaluates from the saved reading.
A requested cycle is not re-read: it skips only the threshold check, still needs phase idle and the idle probe, and so can cycle a small or fresh session on the operator's say.
Factor the "store a reading into State" step shared by `tick`, `tickResuming` and the re-read into one helper so the unknown-means-zero rule is stated once.

In `internal/orchengine/state.go`, `ResetForFreshLaunch` clears `LastContextTokens` and `LastContextKnown`; update its doc comment, which today says the reading survives.

Tests:
in `watcher_test.go`, replay the 2026-10-04 incident: reading 354815 at the last turn end, the fake's reading for that turn end then switching to a compaction boundary with 13673 (the transcript changed without a turn end), soft threshold 300000; past the soft idle with the probe passing, no cycle fires and the saved reading is 13673.
Add a hard-trigger variant whose re-read falls below the cap, and a requested cycle that fires on a small reading with no re-read (the fake records `ContextTokens` calls).
In `state_test.go`, `ResetForFreshLaunch` leaves the reading unknown.

In `internal/orchengine/doc.go`'s idle rules, state the re-read before firing, and that a fresh launch or adopt shows an unknown reading until the new session's first turn end.

**ImpactSummary:** Adds one context re-read per firing attempt in tickIdle and resets the persisted reading on fresh launch and adopt; no seam or signature change.
