# Card 4 — shedrun-alignment-paths

**Create:**
- `internal/shedrun#AlignmentDir`
- `internal/shedrun#AlignmentRel`

**Edit:**
- `internal/shedrun#TestPathConstructors`
- `internal/shedrun#TestDriveReportsRel`
- `internal/shedrun/doc.go`

**Intent:**
`shedrun` declares where the alignment records live (`PATTERN-shed-run-directory`), beside the arbiter's:
- `AlignmentDir(l, runID, point)` is `RunDir(l, runID)/alignment/<point>`, under `_lyx`, so the records are durable (`PATTERN-durable-vs-ephemeral-state`).
- `AlignmentRel(l, runID)` is the anchor-relative `alignment` directory, for the status commit's pathspec in card 8.
- `TestPathConstructors` and `TestDriveReportsRel` gain their rows, and `doc.go` names the new paths beside the arbiter's.

**ImpactSummary:** Two path functions added to `shedrun`; no existing function changes, and no caller exists before cards 8 and 10.
