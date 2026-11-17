# Card 7 — The hub-wide run list for adopt's catch-up

**Create:**
- `plan:internal/battencli#HubRunSlugs` -> `func HubRunSlugs(prime *lymcwd.Location) ([]string, error)`

**Edit:**
- `internal/battencli#TestHubCatchUp`

**Uses:**
- `internal/battencli#DobbyPairSlugs`
- `internal/battencli#NewDobbyParents`
- `internal/battencli#HubCatchUp`

**Intent:**
D6 step 4 runs the catch-up over the whole hub, and `HubCatchUp` takes the run list as a seam; today only a dobby's per-worktree list `DobbyPairSlugs` exists.
`HubRunSlugs` returns, sorted and without duplicates, the union of `DobbyPairSlugs` over the dobby geometry of the prime and of each local beam (`hubgeom.DobbyStart(prime).Beams()`), sharing one `NewDobbyParents(prime)`.
A geometry or list that does not read returns its error, a propagated read error.
Its production caller is adopt's catch-up step (card 9).

`TestHubCatchUp` (tier 1), whose hub already holds runs, gains rows `HubRunSlugsListsTheRunsUnderThePrimeAndEveryLocalBeam` (a run under the prime and one under a beam) and `HubRunSlugsOfAHubWithNoRunListsNone`.

**ImpactSummary:** one new reader beside the catch-up wiring; the existing test gains two rows over its hub.

**Commit:** 7: List every run of the hub for adopt's catch-up
