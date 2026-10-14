# Go driver and tackler in shed, for every FSM-driven run

`shed-tackler` · Note · enhancement, shed, loom, batten

## Seen (2026-10-14)

The Go driver and the tackler landed in `1c812c1f8` for loom runs only.
Their code is in `internal/loomcli` (`drive.go`, `tackler.go`), and the keys `driver_kind`, `tackler`, `tackler_timeout_min` and `tackler_max_spawns` are loom config.
Darn has no `driver_kind`, so a darn run always has an LLM driver.
Batten steps its own shed run in Go but spawns no tackler: an unexpected stop there goes straight to the orch, and batten only reads whether a loom child's tackler or arbiter is in flight to hold its notice.

## Wanted (operator, 2026-10-14)

Any FSM can go wrong, so every shed-driven process needs the tackler, not only loom: darn, batten, and any later recipe.
The tackler, and the Go driver it sits behind, belong in shed as code.
A module's own config can override the tackler's model and budgets, the way loom's keys do today.

## Related

`orch-split` (its "Was `go-driven-shed`" section) wants the same LLM involvement at every level; this note is the shed part of that.
