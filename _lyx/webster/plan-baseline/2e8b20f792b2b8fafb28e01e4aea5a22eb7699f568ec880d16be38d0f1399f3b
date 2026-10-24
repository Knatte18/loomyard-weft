# Card 2 — origin-beam-mark

**Edit:**
- `internal/fabricengine#Origin`
- `internal/fabricengine#TestOrigin_JSONRoundTrip`

**Create:**
- `internal/fabricengine#MarkBeam`
- `internal/fabricengine_test#TestMarkBeam`

**Uses:**
- `internal/fabricengine#WriteOrigin`
- `internal/fabricengine#ReadOriginFor`
- `internal/fabricengine#CommitRecordsPaths`
- `internal/hubforge#AddPair`

**Intent:**
The origin record gains the beam mark (D12): `Origin.Beam bool` with JSON key `beam`, `omitempty`, so a record written before the field reads as no beam and an older binary keeps the key as unknown (PATTERN-tolerant-records).
`MarkBeam` writes `Beam: true` into the origin record of slug's pair, keeping its parent fields, and commits that one path on the pair's records branch with the subject `fabric: mark beam <slug>`; it reports false and writes nothing when the record already carries the mark, so a re-run of the beam verbs is a no-op.
A pair with no origin record is refused naming the slug, since a mark on a missing record would invent its parents, and the way forward tells the two causes apart: a missing records worktree names `lyx fabric reconcile`, which recreates it, and a record file absent from a present records worktree names restoring it from the records history in that worktree, with `git checkout HEAD -- <record path>` while the deletion is uncommitted, else from the commit before the one that deleted it (`git log -1 --format=%H -- <record path>` names that commit, and `git checkout <commit>^ -- <record path>` restores it), then re-running the verb, since reconcile never rewrites a deleted record.
The caller pushes the commit through the pair's detached push (`SpawnPairPush`); `MarkBeam` pushes nothing.
Only `lyx beam create` (through `AddOptions.Beam`, card 4) and `lyx beam promote` (this function) write the mark.
`TestOrigin_JSONRoundTrip` gains a row for a record with the mark and one decoded from JSON without the key.
`TestMarkBeam` over a hubforge hub adds a pair, marks it, reads the record back with `ReadOriginFor` (mark set, parents kept, one new commit), re-runs it (false, no new commit), refuses a pair whose record the test deletes from its present records worktree, naming `git checkout HEAD -- <record path>` in that worktree, and refuses a pair whose records worktree the test removes, naming `lyx fabric reconcile`, each refusal with no commit made.

**ImpactSummary:** One new struct field read as absent by every existing reader, plus a new writer; no caller changes.
