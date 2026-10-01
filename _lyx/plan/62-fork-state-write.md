# Card 62 — fork-state-write

**Edit:**
- `internal/websterengine/audit.go`
- `internal/websterengine/recordbatch.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/doc.go`
- `internal/websterengine/audit_test.go`
- `internal/websterengine/auditseverity_test.go`
- `internal/websterengine/recordbatch_test.go`
- `internal/websterengine/runlevel_test.go`

**Uses:**
- `internal/websterengine/report.go`
- `internal/websterengine/integration.go`
- `internal/websterengine/batchfail.go`
- `internal/websterengine/suspect.go`

**Intent:**
Fix PR #321 round-4 finding C.
No `CheckFork` class covers a fork writing `_lyx/webster/state.json` or anything else under `WebsterDir` but its own report.
A fork that marks its own batch done with the Write tool is refused by `record-batch` as "already terminal" before the audit runs, so neither the done checks nor the fork audit run, and the batch lands.

In `audit.go`:

- A new class `ClassForkStateWrite AuditViolationClass = "fork-state-write"`, which `ClassifyViolation` classes as correctness beside `ClassForkPlanWrite`.
- `planDirSpellings`' body becomes `dirSpellings(dir string) ([]string, error)`, which `planDirSpellings` calls with `geom.PlanDir`;
  `websterDirSpellings(geom Geometry)` calls it with `geom.WebsterDir`.
- `CheckFork(f, outcomePath, summaryPath, workdir string, planDirs, websterDirs []string, ownReport string, fabricRef RefMatcher)`.
  A write within any `websterDirs` spelling that is neither a contract write (which keeps its own finding only) nor `ownReport`, compared in told and link-resolved spellings, yields a `fork-state-write` finding with its `Path` and the detail `fork wrote %q — webster's run state is written only by webster and each fork's own report; a fork writing there can forge a batch record`.
  The doc comment names the fifth class.

Callers:

- `RecordBatch` passes `websterDirSpellings(deps.Geom)` and the batch's own report path.
- `RecordBatch` on a terminal `fork` record no longer refuses first.
  It fetches the session's fork audit once, without the settle wait, and runs `CheckFork` over every transcript not in `SeenForkTranscripts`.
  Any undispositioned correctness finding fails the batch through `failFromFindings`, replacing the terminal record with a failed one and naming the findings' paths as suspects.
  Otherwise nothing is mutated and the existing "already terminal" refusal returns unchanged.
- `runExitAuditCrossCheck` passes `websterDirSpellings(deps.Geom)` and, for each fork, the report of the batch whose `ForkTranscripts` holds its transcript path, or `IntegrationReportPath(deps.Geom.ReportsDir)` when no batch holds it, since only the integration fork runs outside a batch bracket.

`doc.go`'s audit paragraph names the class and the terminal-record audit.

Tests:

- `audit_test.go`'s `TestCheckFork` table gains rows: a write to `<webster>/state.json` → `fork-state-write` with that path;
  a write to the fork's own report → no finding;
  a write to another batch's report → `fork-state-write`;
  a write to `outcome.yaml` → only `fork-contract-write`.
  Every existing call passes the new arguments.
- `auditseverity_test.go`: `fork-state-write` classes as correctness.
- `recordbatch_test.go`: `TestRecordBatch_ForgedTerminalRecordFails`: a fork whose transcript writes `state.json`, with the batch record already terminal `done` on disk → `ErrBatchFailed`, the record terminal `failed` and its reasons naming `fork-state-write`;
  a terminal batch with no new transcript still refuses `already terminal` with `state.json` unchanged.
- `runlevel_test.go`: a run-exit audit over a fork transcript writing `state.json` leaves a pending `fork-state-write` finding, and one writing only its own batch's report leaves none.

**ImpactSummary:** A fork write under webster's run directory other than its own report is a correctness finding at record-batch and at run exit, and record-batch audits a terminal fork batch's new transcripts before refusing it as already terminal.
