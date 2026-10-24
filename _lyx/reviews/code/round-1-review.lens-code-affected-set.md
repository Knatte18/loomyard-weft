```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set — everything outside the touched lines that the diff's changes force.

**Per change, the set found:**

1. `heldTurnEnd.raw` (new field). Covered.
   - Every construction site sets it. A grep for `heldTurnEnd{` finds only the two sites in `pollEventsTick` (`wait.go:851`, `wait.go:872`), and both set `raw`.
   - The held values that `wait.go:859`, `wait.go:1050` and `hold.go:66` handle come from those constructors, so no path carries a nil `raw`.
   - `run.waitingTurnEnd` is stored with `raw` and returned later through `expiredTurnEnd`, so the shell-expiry path also reaches the switch with `raw` set.

2. The `usageLimited` case in the `Wait` switch. Covered.
   - Every held turn end goes through that one switch (`wait.go:325-334`). The `notifyHeld` callers are the switch and `settleDeferredHold` (`hold.go:75`).
   - `settleDeferredHold` only sends `pendingHold`, which the switch sets, so a limited turn end never reaches it.
   - An earlier non-limited deferred hold can still be pending when a limited turn end arrives. Nothing clears it, and it can notify after the limited one. The decision record's bound allows this, and a turn end that arrived in a new turn clears it anyway (`wait.go:300-302`). I count it covered and not a defect.

3. The `UsageLimitReader` interface and `Claude`'s implementation. Covered.
   - `Claude` has the compile-time assertion.
   - `sessionFakeEngine` has the fake and the assertion.
   - Other `Engine` implementers lack the capability and fall back to no limit through the type assertion.
   - The `hold_test.go` row `usageLimit` swaps in `sessionFakeEngine`, which replaces the `session` engine selection. No existing row combines `session` with `usageLimit`, so none breaks.

4. Docs. Covered.
   - `shuttleengine/doc.go` and `claudeengine/doc.go` carry the new behaviour. `doc.go` says the engine's capability is optional and `NotifiedOffset` is untouched, which the code bears out.
   - `PATTERN.md` and `pattern/` need no change. The `rate_limit` literal and the transcript shape stay in `claudeengine`, and no docs or stencils elsewhere describe the hold-notice behaviour.

5. Fixtures and other tests. Covered.
   - `fakes_test.go` adds `usageLimitLine` and the matching method. The fake struct comment at `fakes_test.go:791-794` is updated.
   - `apiErrorEntry` gained an `Error` field. `newestTurnEndError` (`activity.go:~170-190`) and the other users of `apiErrorEntry` ignore the new field, so their behaviour is unchanged.

**Findings:** none.

**Tool calls:**
1. Grepped `heldTurnEnd{` and `UsageLimitReader`/`TurnEndActivity(` across `internal` to find construction sites and other engines/implementers; my context lacked a repo-wide search.
2. Grepped `waitingTurnEnd`, `notifyHeld(` and the docs/PATTERN for the hold notice, to find other notify paths; my context lacked this.
3. Read `wait.go:853-868` and `wait.go:1036-1062` to see how `waitingTurnEnd` becomes a held return, since the manifest showed only part of that code.
