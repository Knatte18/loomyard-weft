# Card 7 — The hub-wide run list for adopt's catch-up

**Create:**
- `internal/battencli#HubRunSlugs`

**Edit:**
- `internal/battencli#TestHubCatchUp`
- `cmd/lym#testOnlyCodeAllowed`

**Uses:**
- `internal/battencli#DobbyPairSlugs`
- `internal/battencli#NewDobbyParents`
- `internal/battencli#HubCatchUp`

**Intent:**
D6 step 4 runs the catch-up over the whole hub, and `HubCatchUp` takes the run list as a seam; today only a dobby's per-worktree list `DobbyPairSlugs` exists.
`HubRunSlugs` returns, sorted and without duplicates, the union of `DobbyPairSlugs` over the dobby geometry of the prime and of each local beam (`hubgeom.DobbyStart(prime).Beams()`), sharing one `NewDobbyParents(prime)`.
A geometry or list that does not read returns its error, a propagated read error.
Its production caller is adopt's catch-up step (card 9).
Until then no main reaches it, so `cmd/lym`'s `testOnlyCodeAllowed` gains `internal/battencli.HubRunSlugs`, its Why naming card 9, as the `internal/runfsm.Order` entry does; the exact key is the one the guard reports at this card's tree.

`TestHubCatchUp` (tier 1), whose hub already holds runs, gains rows `HubRunSlugsListsTheRunsUnderThePrimeAndEveryLocalBeam` (a run under the prime and one under a beam) and `HubRunSlugsOfAHubWithNoRunListsNone`.

**ImpactSummary:** one new reader beside the catch-up wiring, with a test-only-code allowlist entry until card 9; the existing test gains two rows over its hub.

**Commit:** 7: List every run of the hub for adopt's catch-up
