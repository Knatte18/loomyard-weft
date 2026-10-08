# Card 4 — Export the board-presence predicate

**Rename:**
- `internal/preflight#boardLyxPresent` -> `internal/preflight#BoardLyxPresent`

**Intent:**
Card 8's `hubgeom.ReconcileGeometry` must decide "is this a hub" with the same stat `preflight.HubPresent` and `preflight.ResolveMode` use, and the predicate's own doc comment says it exists so those callers never grow a second copy of the check.
Export it under the same name, capitalized, so the third caller shares it.

The rename updates its two callers in `predicates.go`, `HubPresent` and `ResolveMode`, and the doc comment's opening self-reference; its body and behavior are unchanged.

**Commit:** 4: preflight: export the board-presence predicate
