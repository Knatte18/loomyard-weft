# Card 17 — one-describe

**Create:**
- `plan:internal/loomcli#commitDescription` -> `func commitDescription(location *lymcwd.Location) error`

**Edit:**
- `internal/loomcli#darnDeps`
- `internal/loomcli/doc.go`
- `internal/landingshed#DescribeInputs`
- `internal/landingshed#DescribeTable`
- `internal/landingshed#runRecordBullets` -> `func runRecordBullets(in DescribeInputs) (string, error)`
- `internal/landingshed#TestDescribeSpec_EmptyFieldIsError`
- `internal/landingshed#TestDescribeSpec_RunRecordsOldestFirstLiveLast`
- `internal/landingshed/doc.go`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.wireDarn`
- `contracts/stencils/landing/landing-template-describe.md`
- `contracts/stencils#entries`
- `contracts/stencils#roleOpeningStencils`
- `contracts/stencils#TestStencils_TopLevelMarkers`

**Delete:**
- `internal/landingshed#DarnDescribeTable`
- `internal/landingshed#DarnDescribeInputs`
- `internal/landingshed#darnDescribeStencilName`
- `internal/landingshed#TestDarnDescribeSpec`
- `contracts/stencils#LandingTemplateDescribeDarn`
- `contracts/stencils/landing/landing-template-describe-darn.md`

**Intent:** One Describe table and one stencil serve every loom recipe (D8). The stencil reads the decision record, the diff and the run records that exist, rendering an absent one as `(none)`, and `landing-template-describe-darn.md` goes.

- `DescribeInputs.RunRecordPath` becomes optional, like `PriorRunRecordPaths`: `runRecordBullets` renders one bullet per record that exists on disk, oldest first and the live one last, skipping a path that does not exist, and the single line `  - (none)` when none exists. A stat error other than not-exist is returned from `DescribeTable`, which no longer refuses an empty `RunRecordPath`; every other field stays required.
- `landing-template-describe.md` reads, in order: the decision record, whose `## Goal` is the task's scope as amended by its `Added after Discussion` entries, and whose carry-over entries under `## Open risks` become manual-check items (a Plan-Review or Code-Review entry, never a Discussion-Review one); the board entry via `lym board get`; the diff; and the run records, only as a source of manual-check items, `(none)` meaning the run has none. It says the change may have been fixed after an earlier description was written, so it describes the diff as it stands.
- The wiring builds one table for every loom recipe: `wire`'s `DescribeTable` closure passes `summaryparser.Path(websterGeom.WebsterDir)` and the archived generations' summaries unconditionally, and its own stat loop over the prior records goes, so a darn run, which has no webster summary, renders `(none)`. `wireDarn` stops replacing `DescribeTable`, and its review-subdir lookup goes with the darn table, the record's carry-over entries carrying the perch's open findings instead.
- `DarnDescribeTable`, `DarnDescribeInputs`, `darnDescribeStencilName`, the embedded `LandingTemplateDescribeDarn` and its registry and role-opening entries go.
- `commitDescription` commits the landing directory with the change-description subject, a repeat commit over an unchanged directory a no-op; `wire`'s `CommitDescription` and `darnDeps`' `Commit` both call it, so every loom recipe has one description commit (D8). It extracts the two closures' identical bodies, adding no behaviour, and gains no test of its own.

Tests: `TestDescribeSpec_RunRecordsOldestFirstLiveLast` writes its records to a temporary directory and gains a row whose live and prior paths do not exist, rendering `(none)`, and a row whose prior path exists and live path does not, rendering the prior alone; `TestDescribeSpec_EmptyFieldIsError` drops its `RunRecordPath` row; `TestDarnDescribeSpec` goes; `TestStencils_TopLevelMarkers` drops the darn Describe stencil's row. The landingshed doc's Describe paragraph names one table.

**ImpactSummary:** Two Describe tables and stencils become one, and the two description commits one function; the darn wiring stops overriding the table, and only the stencil registry and landingshed tests also change.
