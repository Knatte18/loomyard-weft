# Card 13 — land-blockers

**Create:**
- `plan:internal/battencli#BeamLandBlockers` -> `type BeamLandBlockers struct`
- `plan:internal/battencli#BeamLandBlockers.Empty` -> `func (b BeamLandBlockers) Empty() bool`
- `plan:internal/battencli#ReadBeamLandBlockers` -> `func ReadBeamLandBlockers(prime *lyxcwd.Location, slug string) (BeamLandBlockers, error)`
- `plan:internal/battencli#BeamLandMember` -> `type BeamLandMember struct`

**Edit:**
- `internal/battencli#beamMembers` -> `func beamMembers(prime *lyxcwd.Location, slug string) ([]BeamLandMember, error)`
- `internal/battencli#BeamLandRefusal`
- `internal/battencli#beamLandRefusal` -> `func beamLandRefusal(slug string, blockers BeamLandBlockers) error`
- `internal/battencli#TestBeamLandRefusal`
- `internal/beamcli#landCmd`
- `internal/beamcli/doc.go`

**Delete:**
- `internal/battencli#beamOrchLive`
- `internal/battencli#beamMember`

**Intent:**
D10: `BeamLandRefusal` loses its orch and watcher blocker, since the beam's orch runs until Beam-Down, so every caller (`lyx beam land`, Beam-Await's re-check, Finalize, `lyx batten cancel <beam>` and the dobby's can-land check) judges members and member pairs alone; and its facts become readable as data, since Finalize words its own ways forward from them (D9).

- `ReadBeamLandBlockers(prime, slug)` reads what `BeamLandRefusal` judges: the open members, through `beamMembers`, and the local pairs forked from the beam, through `beamChildPairs`.
  The unexported `beamMember` gives way to the exported `BeamLandMember`, which loomcli (card 17) and beamcli (card 23) read: `Slug`, `HasPair` and `Running`, whether the member's board status `IsRunStatus` accepts; `beamMembers` returns it, `BeamLandBlockers.Members` holds it, `ChildPairs` lists the forked pairs and `Empty` reports that neither holds anything.
- `BeamLandRefusal` is `beamLandRefusal(slug, ReadBeamLandBlockers(...))`: every member and pair blocker words its way forward as today, and the orch and watcher cases go with `beamOrchLive`, so `battencli` stops reading the beam's orch state.
- `landCmd`'s help and `beamcli/doc.go`'s list of the refusal's blockers drop the orch line.

`TestBeamLandRefusal` drops its orch and watcher rows, keeps its member and pair rows over `BeamLandBlockers`, and its existing passing row runs over empty `BeamLandBlockers`, which reaches `Empty`; a member row asserts `Running` carried through.

**ImpactSummary:** the refusal's one judgment loses a blocker and gains a data form; its callers keep calling `BeamLandRefusal`, and the only removed reader, the orch liveness read, had no other caller.

**Commit:** 13: A beam's landing refusal judges members and member pairs alone, readable as blockers
