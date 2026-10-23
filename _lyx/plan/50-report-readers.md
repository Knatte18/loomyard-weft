# Card 50 — report-readers

**Create:**
- `internal/reportengine#retiredRowNames`
- `internal/reportengine#TestReadSegments_CodeAndRetiredSubdirs`

**Edit:**
- `internal/reportengine#loomRowPhases`
- `internal/reportengine#darnRowPhases`
- `internal/reportengine#darnPhases`
- `internal/reportengine#recipePhaseTables`
- `internal/reportengine#runWaits`
- `internal/reportengine#settlerOf`
- `internal/reportengine#RunRow.classifyBatten`
- `internal/reportengine#RunRow.readSegments`
- `internal/reportengine#TestComputeTiming`
- `tools/tokencount/profile.go`
- `tools/tokencount/main.go`

**Extends:**
- `internal/reportengine/doc.go`

**Intent:** `lyx report convergence` and `lyx report timing` read runs recorded before and after the change (D13).
The report names the retired rows by the literals `retiredRowNames` declares (Publish, PR-Align, PR-Gate, PR-Rework, Merge-Bouncer, Merge-Burler-Review, Merge-Burler-Fix and the three Webster-Review rows), so it stops importing their `loomshed` constants, which the next card deletes.
The timing phase tables keep the retired rows in their phases for old runs and add Code-Bouncer, Code-Burler-Review and Code-Burler-Fix to the webster phase and PR-Review to the publish-finalize phase; the PR-Gate wait and stop readers and the batten classifier's rules read PR-Gate and the retired verify rows by literal, so an old run still classifies as it did.
`readSegments` reads the `code` review run directory as the Code-Review segment and an old run's `webster` directory as the same segment, and drops the `merge` directory, so the Merge-Review figures go and Code-Review is read where Webster-Review was; the PR-Rework figures go with the rework rows' phase entries no longer counting a new run's time.

`tools/tokencount` reads a run's code review files from the `code` review directory, and an old run's from `webster`, where its constant and its comments named the Webster-Review directory alone.

`TestReadSegments_CodeAndRetiredSubdirs` pins a run with a `code` directory, an old run with `webster` and `merge` directories (`webster` read as Code-Review, `merge` dropped), and the timing test gains a new-recipe run whose PR-Review and Code rows land in their phases and an old run whose PR-Rework rows still land in the publish-finalize phase while no new-recipe run reports a PR-Rework span.

`doc.go`'s phase and segment paragraphs follow.

**ImpactSummary:** The report's row tables and segment reader change; old runs keep reading through literals, and no other package reads them.
