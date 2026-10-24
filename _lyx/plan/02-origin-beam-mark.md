# Card 2 — origin-beam-mark

**Edit:**
- `internal/fabricengine#Origin`
- `internal/fabricengine#TestOrigin_JSONRoundTrip`

**Create:**
- `plan:internal/fabricengine#MarkBeam` -> `func MarkBeam(rec *Mutations, l *lyxcwd.Location, slug string) (bool, error)`
- `plan:internal/fabricengine_test#TestMarkBeam` -> `func TestMarkBeam(t *testing.T)`

**Uses:**
- `internal/fabricengine#WriteOrigin`
- `internal/fabricengine#ReadOriginFor`
- `internal/fabricengine#CommitRecordsPaths`
- `internal/hubforge#AddPair`

**Intent:**
The origin record gains the beam mark (D12): `Origin.Beam bool` with JSON key `beam`, `omitempty`, so a record written before the field reads as no beam and an older binary keeps the key as unknown (PATTERN-tolerant-records).
`MarkBeam` writes `Beam: true` into the origin record of slug's pair, keeping its parent fields, and commits that one path on the pair's records branch with the subject `fabric: mark beam <slug>`; it reports false and writes nothing when the record already carries the mark, so a re-run of the beam verbs is a no-op.
A pair with no origin record is refused naming the slug and `lyx fabric reconcile`, since a mark on a missing record would invent its parents.
The caller pushes the commit through the pair's detached push (`SpawnPairPush`); `MarkBeam` pushes nothing.
Only `lyx beam create` (through `AddOptions.Beam`, card 4) and `lyx beam promote` (this function) write the mark.
`TestOrigin_JSONRoundTrip` gains a row for a record with the mark and one decoded from JSON without the key.
`TestMarkBeam` over a hubforge hub adds a pair, marks it, reads the record back with `ReadOriginFor` (mark set, parents kept, one new commit), re-runs it (false, no new commit) and refuses a slug with no record naming `lyx fabric reconcile`.

**ImpactSummary:** One new struct field read as absent by every existing reader, plus a new writer; no caller changes.
