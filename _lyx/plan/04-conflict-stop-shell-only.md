# Card 4 — Landing's conflict-session stop removes a present shell-only strand

**Edit:**
- `internal/loomcli#conflictSessionStopper`
- `internal/loomcli#TestConflictSessionStopper`

**Intent:**
D1's Consequences: under the new judgment a conflict strand whose engine exited to its shell reads `Live: false` while its pane is present, and `conflictSessionStopper` skips every strand that is not live, so it would leave that strand's pane and record.
The stopper now stops a non-retiring conflict strand that is live, or own-window and present, the same rule shuttle's `strandRemovable` applies (internal/shuttleengine/stop.go), so the stop it calls (`Runner.StopStrand`) finds the strand removable.
A strand that is neither live nor present (its pane gone) is still skipped, as is a retiring one.
The doc comment's "every live, non-retiring strand" becomes that rule.

`TestConflictSessionStopper`'s `conflict` helper gains the own-window and present flags, and the table gains a row where a conflict strand with `Live: false, OwnWindow: true, Present: true` is stopped, and keeps "dead conflict strand stops nothing" as a strand with `Live: false` and no present pane.
A row with a conflict strand `Live: false, OwnWindow: false, Present: true`, a window 0 pane reconcile kills at the next reed op, stops nothing, as shuttle's rule leaves it.
The test's doc comment is updated to the same rule.

**ImpactSummary:** One predicate in one closure with its one production caller in sharedbootstrap.go unchanged; the stop it calls already removes such a strand.

**Commit:** 4: landing's conflict stop removes a conflict strand whose engine exited
