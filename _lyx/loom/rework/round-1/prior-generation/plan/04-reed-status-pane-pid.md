# Card 4 — reed-status-pane-pid

**Edit:**
- `internal/reedengine#StrandStatus`
- `internal/reedengine#Engine.Status`
- `internal/reedengine#TestStatus_ReportsOwnWindowStrandsFromTheSessionListing`

**Intent:** Let a reader learn each strand's pane process from reed's read-only status query of the hub's tmux server (D1), which creel's identity walk matches peers against.
`StrandStatus` gains a `PanePID int` field: the pane's process id from the same `list-panes` listing `Status` already reads (`LivePane.PID`), 0 for a strand whose pane is in no listing or is dead.
`Status` fills it from the present panes it already computes; it runs no new tmux command and stays read-only.
`TestStatus_ReportsOwnWindowStrandsFromTheSessionListing`, whose fixture already lists live and dead panes with their pids, asserts `PanePID` 4323 for the strand bound to `%2`, 4325 for the live own-window strand on `%5`, 0 for the strand on the dead pane `%6` and 0 for the strand on `%9`, which no listing holds; its doc comment gains the pane pid.

**ImpactSummary:** One additive field on StrandStatus filled by Status from the listing it already reads; every existing reader ignores it.
