# Card 13 — logger-sweep-dry-run

**Create:**
- `internal/logger#planSweep`
- `internal/logger#SweepDir`
- `internal/logger#LogsDirAt`
- `internal/loggerconfig#LoadOrDefault`
- `internal/logger#TestSweepDir_DryRunListsWhatApplyRemoves`

**Edit:**
- `internal/logger#sweep`
- `internal/logger#LogsDir`
- `cmd/lyx#exitSweepBounds`
- `internal/logger_test#TestLogsDir_AnchorsAtAnchorPath`
- `internal/loggerconfig_test#TestLoad_ResolvesBounds`
- `internal/loggerconfig_test#TestLoad_InvalidValueErrorsNamingKey`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/logger#Sweep`
- `internal/loggerconfig#Load`

**Intent:**
Give logger's retention sweep the dry-run shape `lyx cleanup`'s log segment needs, and move the exit sweep's bound choice where that segment can call it (D7 segment 5).

`planSweep` is `sweep`'s decision half: it groups and ranks exactly as today and returns the paths the policy deletes, deleting nothing.
`sweep` becomes `planSweep` followed by the deletions, keeping its signature and its tolerance of a failed delete, so `Sweep` and every existing `TestSweep_*` test are unchanged.
`SweepDir(dir, bounds, apply)` returns `planSweep`'s list at `time.Now()` with `os.Stat`, deleting each path, a failed delete tolerated as `Sweep` tolerates it, only when `apply` is true; a dry run and an apply therefore list the same paths.
`TestSweepDir_DryRunListsWhatApplyRemoves` seeds aged and fresh trace groups, asserts a dry run lists the aged group's files and leaves every file in place, and that an apply then lists the same paths and removes exactly them.

`LogsDirAt(anchorPath)` is the logs directory under an anchor; `LogsDir(l)` returns `LogsDirAt(l.AnchorPath())`, so the cleanup module can name `_board`'s logs directory without a `Location` and without spelling `.lyx` or `logs` itself.
`TestLogsDir_AnchorsAtAnchorPath` also asserts `LogsDir(l) == LogsDirAt(l.AnchorPath())`.

`LoadOrDefault(anchorPath)` is the bound choice `exitSweepBounds` makes for a sink that is not redirected: `Load`'s bounds, or, when `Load` fails, one `logger.Warn` naming `ConfigPath(anchorPath)` and the error, and `logger.DefaultRetentionBounds()`.
`exitSweepBounds` keeps its redirected-sink branch and calls `LoadOrDefault` for the other, with the same Warn text, so `TestExitSweepBounds` passes unchanged.
`TestLoad_ResolvesBounds` also asserts `LoadOrDefault` returns the same bounds as `Load` for each row, and `TestLoad_InvalidValueErrorsNamingKey` that it returns the defaults for each invalid row.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/logger` row grows by one.

**ImpactSummary:** sweep splits into plan and delete with its contract kept; exitSweepBounds delegates to loggerconfig with the same Warn and fallback.
