# Card 9 — beam-status-adopt

**Create:**
- `plan:internal/beamcli#adoptSection` -> `type adoptSection struct`
- `plan:internal/beamcli#adoptSectionOf` -> `func adoptSectionOf(record adoptrecord.Record, state string) *adoptSection`
- `plan:internal/beamcli#TestAdoptSectionOf` -> `func TestAdoptSectionOf(t *testing.T)`

**Edit:**
- `internal/beamcli#statusResult`
- `internal/beamcli#dobbyStatus`
- `internal/beamcli_test#TestStatus_PrintsThePrimesDobbyAndEachLocalBeams`

**Extends:**
- `internal/beamcli#statusCmd`
- `internal/beamcli/doc.go`

**Uses:**
- `plan:internal/adoptrecord#Read`
- `plan:internal/adoptrecord#Record.ReadState`
- `plan:internal/adoptrecord#Request.Command`
- `plan:internal/adoptrecord#WorkerAlive`

**Intent:**
D9: `lym beam status` shows an unfinished adopt.
`statusResult` gains `Adopt *adoptSection` (`adopt`, omitted when nil).
`dobbyStatus` reads the hub's record with `adoptrecord.Read` and its state with `ReadState(adoptrecord.WorkerAlive)`.
A read error joins the verb's failures like an adopt check's.

`adoptSectionOf` returns nil for a `done` record.
Otherwise the section holds:
- the subjects;
- the state: `running`, `interrupted` or `stopped`;
- the step in flight, or the step stopped at;
- the steps done;
- the way forward:
  - `running`: the re-run of the same `lym adopt`, `Request.Command`, which follows the worker;
  - `interrupted`: that re-run, which resumes it;
  - `stopped`: the stop's own way forward;
  - and, for `interrupted` and `stopped`, `lym adopt --discard`.
It also holds a rendered `lines` text opening `adopt of <subjects>:`.
The read takes no lock and creates nothing, so the verb still changes nothing.

`statusCmd`'s `Long` text and the beamcli `doc.go` gain the `adopt` section.

Tests:
- `TestAdoptSectionOf` (tier0) has one row per state.
  Running names its step in flight and the re-run.
  Interrupted names the re-run and `lym adopt --discard`.
  Stopped names its stop's way forward and `lym adopt --discard`.
  Done gives nil.
- `TestStatus_PrintsThePrimesDobbyAndEachLocalBeams` (tier1) asserts no `adopt` field with no record.
  After writing an `adopt.json` that reads `stopped` at `deploy`, it asserts one `adopt` section with the record's state and step;
  `TestAdoptSectionOf` pins the way-forward wording.
  The `check.json` it writes still shows as the one `adopt_checks` item.
  An `adopt.json` that does not decode then fails the verb with an error naming `lym adopt --discard`, while the dobbies and `adopt_checks` still read.

**ImpactSummary:** one new optional field in the `beam status` envelope, read lock-free; no other verb or package changes.
