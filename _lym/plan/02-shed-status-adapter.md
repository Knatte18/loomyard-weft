# Card 2 — shed-status-adapter

**Create:**
- `internal/shedengine#StatusLastMachine`
- `internal/shedengine#StatusRecorded`
- `internal/shedengine#TestStatusCodec`

**Uses:**
- `internal/shedengine#ParseStatusTolerant`
- `internal/shedengine#Status`

**Extends:**
- `internal/shedengine/doc.go`

**Intent:**
Shed's half of the status codec seam D13 names: the verdicts fabric's push and origin concern takes from a status record's bytes.
Today `readCodeTipAt` (`internal/fabricengine/originreads.go`) parses the record with `shedengine.ParseStatusTolerant` and reads `History[len-1].Machine`,
and `recordedRuns` and `recordedPairs` decode each run's status record to learn whether the run is recorded.

`StatusLastMachine` parses `data` with `ParseStatusTolerant` and returns the `Machine` of the last `History` entry, or "" when the history is empty.
`StatusRecorded` parses `data` with `ParseStatusTolerant` and reports true when it parses.
Each returns a parse error wrapped, naming the status record.
They take and return only bytes, a string and a bool, so `internal/shedengine` keeps its import allowlist (`internal/shedengine#TestProducerSeamInvariant_AllowlistOnly`) and imports nothing of fabric.

`TestStatusCodec` is a tier0 table test in package `shedengine`.
Each row writes a `Status` through `state.UpdateJSON` over shed's `Status` type at a temp path, the write every shed status writer makes (`internal/shedengine/run.go`, `entry.go`), since shed has no single named writer;
it reads the file's bytes back and asserts both verdicts:
two history entries name the second's machine and read recorded; no history reads "" and recorded; an unknown key in the file is tolerated by both; bytes that are no JSON return an error from both.
It adds under 1 second to one tier0 run.

The doc.go addition says the package supplies the status codec fabric's required seam takes, and that fabric holds only bytes.

**ImpactSummary:** Adds two exported decoders and their test to shedengine; no existing member changes and the import allowlist holds.
