# run-time-by-role: tokencount time columns, idle row and Producers tables

Batch 01 (`01-shedrun-tree-records`) ran all six cards and recorded `done` at head `a903dcd6870d4e22ccc956b9bc756a5aa71a57fe`.
The fork committed one commit per card:

- Card 1 added shedrun's tree-relative seed and status paths and a seed decoder over bytes.
- Card 2 measured turn time per role and added time columns and an `idle` row to the role tables.
- Card 3 read each archive tag's commit and commit date, newest first per slug.
- Card 4 set a run's span from its archived shed status and added the Producers tables.
- Card 5 wired the status read into run, refused a foreign `-hub` without `-weft`, and retitled the report.
- Card 6 described the time columns, idle row, span and Producers table in the package doc and launchers.

Deviations from the declared file-ops, both informational:

- `tools/tokencount/orch_test.go` changed because its expected table rows needed the new time columns and the `idle` row.
- `tools/tokencount/shed.go` is a new file holding the shed record, Producers and span code, since the cards named symbols but no file.

`record-batch` also returned informational scope-outside-plan warnings for symbols in `internal/shedrun` and `tools/tokencount` touched outside the batch's own target glyphs.
The fork's report says it edited the test-budget file with `sed`, against the prompt's editing rule, without changing anything outside the intended edits.

## Verify gate fixes

The gate's first attempt failed on `TestEnforcement_FabricVocabulary/tree-scan`, which found a bare weft token outside the owner set in `tools/tokencount/main_test.go` and `tools/tokencount/time_integration_test.go`.
The fixer fork renamed that token to `records` in both files, covering identifiers, a string literal and comments.
Its commits:

- `5948e3769` — `fix: tokencount tests drop the bare weft token the fabric-vocabulary scan bans`
- `4445e8ae8` — `fix: gofmt alignment in the main_test.go struct, which the rename had thrown off`

The fork ran `lyx gate test --tags integration ./internal/lyxcwd ./tools/tokencount` after the first commit and it exited 0.
It did not re-run after the whitespace-only gofmt commit.
