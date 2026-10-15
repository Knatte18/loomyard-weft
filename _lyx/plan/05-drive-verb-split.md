# Card 5 — drive-verb-split

**Move:**
- `internal/loomcli#loomCLI.driveCmd`
- `internal/loomcli#driveEnvelopePath`
- `internal/loomcli#driveRefusal`

**Intent:**
Move the three loom-only members out of `internal/loomcli/drive.go` into `internal/loomcli/drivewiring.go`, so card 6 can move `drive.go` whole into `internal/sheddriver` while `lyx loom drive` stays loom's verb (D3).
`driveCmd` builds the hidden `drive` cobra verb, `driveEnvelopePath` composes a saved stop envelope's path under `shedrun.DriveReportsDir`, and `driveRefusal` is the loom refusal text of a second Go driver, quoted by `refusal-spec.md`'s "drive: a Go driver already drives the run" row; all three stay in `loomcli`, unchanged, and `drive.go`'s file header stops describing "the verb that wires the real ones sits beside it".
A same-package move with no behaviour change; the existing `drive` tests are the coverage.
