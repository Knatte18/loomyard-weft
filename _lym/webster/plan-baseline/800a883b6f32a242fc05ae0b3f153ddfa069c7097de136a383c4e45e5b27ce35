# Card 5 — gate-holder-marker

**Create:**
- `plan:internal/gatecli#runMarkerEntry` -> `func runMarkerEntry(pid int) string`

**Edit:**
- `internal/gateslot#Holder`
- `internal/gateslot/doc.go`
- `internal/gatecli#runTest`
- `internal/gatecli#acquireSlot`
- `internal/gateslot#TestHolders_NamesLiveHoldersAndDropsRecordsOfFreeLocks`
- `internal/gatecli#TestGateTest_DetachedRun`

**Uses:**
- `internal/gatecli#enclosingGateEnv`
- `internal/gatecli#newDetachRig`
- `internal/gatecli#newRelease`
- `internal/gatecli#releaseEnv`
- `internal/gatecli/doc.go`

**Intent:**
A tier run records its environment marker in its slot holder record, so the gate-hold notice (card 6) can name a process outside the holder's tree (D6).

`gateslot.Holder` gains `Marker string` (`yaml:"marker,omitempty"`): the `NAME=value` entry every process the tier run starts carries in its environment, empty for a holder that is no tier run.
The record is decoded without `KnownFields`, so an older record reads with no marker.
`gateslot`'s package doc, where it describes the holder record, says a tier run's record carries that marker.

`runMarkerEntry` returns `enclosingGateEnv + "=" + strconv.Itoa(pid)`.
`runTest` forms the entry it appends to `exe.env` through it, and `acquireSlot` sets `Marker: runMarkerEntry(os.Getpid())` in the `gateslot.Holder` it acquires with, so the record and the environment carry the same entry.
The other holders (redundancy, mutants, test map) keep no marker.
Bound: a nested `lym gate test` inside the holder's slot takes no slot and records no holder, and it replaces the entry with its own PID for every process it starts, so a process a nested run started carries the nested run's marker, and the gate-hold notice does not name it as a marked process outside the tree.

Tests:
`TestHolders_NamesLiveHoldersAndDropsRecordsOfFreeLocks` acquires with a `Marker` and asserts `Holders()` returns it; its stale record, which has no `marker` key, is the older-record shape, and a second live record written without the key reads with an empty marker.
`TestGateTest_DetachedRun`'s first subtest already holds a tier run going through `newRelease` and `releaseEnv` before it calls `release()`.
Before that call it reads the rig's pool `Holders()` and asserts the tier holder's `Marker` equals `LYM_GATE_TEST_PID=<the holder's PID>`.

**ImpactSummary:** One optional field on the holder record and one helper in `gatecli`; `lym gate status` and every other `Holders` reader ignore the new field.

**Commit:** 5: Record a tier run's environment marker in its gate slot holder record
