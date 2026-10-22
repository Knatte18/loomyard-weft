# Card 14 — tender-status-owners

**Edit:**
- `internal/tenderengine#Deps`
- `internal/tenderengine#StatusReport`
- `internal/tenderengine#Status`
- `internal/tenderengine#TestStatus`
- `internal/tendercli#HubDeps`
- `internal/tendercli#statusCmd`
- `internal/tenderengine/doc.go`
- `docs/overview.md`

**Create:**
- `plan:internal/tenderengine#PairOwner` -> `type PairOwner struct`
- `plan:internal/tendercli#pairOwners` -> `func pairOwners(owners boardengine.Owners, slugs []string, here string, parentOf func(slug string) (string, error)) ([]tenderengine.PairOwner, error)`
- `plan:internal/tendercli#TestPairOwners` -> `func TestPairOwners(t *testing.T)`

**Uses:**
- `plan:internal/boardengine#ReadOwners`
- `plan:internal/boardengine#Owners.Effective`
- `plan:internal/boardengine#OwnedElsewhere`
- `plan:internal/fabricengine#ParentSlug`
- `plan:internal/machinename#Name`

**Intent:**
D9 for `lyx tender status`, so the orch never reads a run owned elsewhere as stalled or landed.
`PairOwner` holds `Slug`, `Owner`, `Local` (this machine holds the pair) and `OwnedHere`.
`Deps` gains `Owners func() ([]PairOwner, error)`, and `Status` fills the new `StatusReport.Owners` from it; a nil seam reports none, and a seam error is reported inside the part as `OwnersError`, leaving the rest of the report intact, as a failed process reading is.
`pairOwners` lists each local pair of `slugs` with its effective owner, then each run-held board entry owned by another machine that has no local pair, as owned by that machine with `Local` false.
`HubDeps` fills `Owners` by reading the hub board with `boardengine.ReadOwners`, this machine's name and `fabricengine.ParentSlug`, over the pair slugs it already lists; `statusCmd`'s help, `tenderengine`'s doc's Status section and `docs/overview.md`'s tender entry name the owners part.
`TestStatus` gains rows: the owners part carries the seam's answer, and a seam error lands in `OwnersError` with the rest of the report intact.
`TestPairOwners` covers a local pair owned here, a local pair owned elsewhere, a run-held entry owned elsewhere with no local pair, a run-held entry owned here with no local pair (left out), an unowned hub, and a `parentOf` error returned.

**ImpactSummary:** tender's status read only; pause, resume and revive are untouched.

**Commit:** 14: tender: status names each pair's owner and the runs owned elsewhere
