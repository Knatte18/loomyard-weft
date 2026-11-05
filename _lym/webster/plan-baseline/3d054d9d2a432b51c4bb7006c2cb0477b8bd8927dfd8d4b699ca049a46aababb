# Card 18 — record-write-stored

**Edit:**
- `internal/loomshed#RecordWriteDeps`
- `internal/loomshed#recordWrite.Call`
- `internal/loomshed#TestRecordWrite_Call`
- `internal/loomcli#recordWriteDeps`
- `internal/shedrecipe#recordWriteEntry`
- `internal/shedrecipe#TestRecordWriteEntry`

**Uses:**
- `internal/loomshed#StoredRecordSupportLog`
- `internal/loomshed#BoardRecord`
- `internal/loomcli#seedDiscussedRecord`
- `internal/boardengine#Board.DecisionRecord`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:** D3: Record-Write seeds a discussed entry's stored record.
`RecordWriteDeps` gains `ReadStored func() (record []byte, found bool, err error)`, which returns the run's board entry's stored decision record when the entry is flagged discussed and its record file exists.
`recordWrite.Call`, when the record is missing, calls `ReadStored` first: a found record is written verbatim as the run's decision record and a missing support log is written as `StoredRecordSupportLog()`; otherwise it renders `BoardRecord(body)` and `boardSupportLog()` from the entry body as today.
A file that exists is never overwritten, as today, so a resume keeps every entry added after the record was written.
This holds for every recipe with a Record-Write row: darn takes a discussed entry's record as final, raddle takes the Sketch's, and pawl a raddle-filed project's record as its beam's project record.
`loomcli.recordWriteDeps` fills `ReadStored` through `boardengine.OpenHub` and `Board.DecisionRecord` of the run's slug, the read `seedDiscussedRecord` makes.
`recordWriteEntry` requires the seam with `requireSeam`, as it does `ReadBody`.
`TestRecordWrite_Call` gains rows: a discussed entry (the stored record written verbatim, the stored-record support log written, `ReadBody` not called); an undiscussed entry, and one flagged discussed whose record file is missing, both read as not found (the body rendered as before); a `ReadStored` error (a returned error naming the step); a record already present (neither read).
`TestRecordWriteEntry` gains the unset `ReadStored` refusal row.
`TestWire_EverySeamFilled` checks `loomcli` fills the new field.

**ImpactSummary:** Record-Write's missing-record branch gains a first read; an undiscussed entry's path is unchanged, and the one wiring site and one registry entry move with the new seam.
