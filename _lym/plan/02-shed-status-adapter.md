# Card 2 — shed-status-adapter

**Create:**
- `plan:internal/shedengine#StatusLastMachine` -> `func StatusLastMachine(data []byte) (string, error)`
- `plan:internal/shedengine#TestStatusLastMachine` -> `func TestStatusLastMachine(t *testing.T)`

**Uses:**
- `internal/shedengine#ParseStatusTolerant`
- `internal/shedengine#Status`

**Extends:**
- `internal/shedengine/doc.go`

**Intent:**
Shed's half of the status codec seam D13 names: the one verdict fabric's push and origin concern takes from a status record's bytes.
Today `readCodeTipAt` (`internal/fabricengine/originreads.go`) parses the record with `shedengine.ParseStatusTolerant` and reads `History[len-1].Machine`.

`StatusLastMachine` parses `data` with `ParseStatusTolerant` and returns the `Machine` of the last `History` entry, or "" when the history is empty.
A parse error is returned wrapped, naming the status record.
It takes and returns only bytes and a string, so `internal/shedengine` keeps its import allowlist (`internal/shedengine#TestProducerSeamInvariant_AllowlistOnly`) and imports nothing of fabric.

`TestStatusLastMachine` is a tier0 table test in package `shedengine`.
Each row writes a `Status` through `state.UpdateJSON` over shed's `Status` type at a temp path, the write every shed status writer makes (`internal/shedengine/run.go`, `entry.go`), since shed has no single named writer;
it reads the file's bytes back and asserts the machine:
two history entries name the second's machine; no history reads ""; an unknown key in the file is tolerated; bytes that are no JSON return an error.
It adds under 1 second to one tier0 run.

The doc.go addition says the package supplies the status codec fabric's required seam takes, and that fabric holds only bytes.

**ImpactSummary:** Adds one exported decoder and its test to shedengine; no existing member changes and the import allowlist holds.
