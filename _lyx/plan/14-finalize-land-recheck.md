# Card 14 — finalize-land-recheck

**Create:**
- `plan:internal/landingshed#LandBlocker` -> `type LandBlocker struct`
- `plan:internal/landingshed#LandHaltPrefix` -> `const LandHaltPrefix = "land check: "`
- `plan:internal/landingshed#Finalize.landCheck` -> `func (fz *Finalize) landCheck(ctx context.Context) (shedengine.Outcome, shedengine.OutputPointer, error, bool)`
- `plan:internal/landingshed#landHaltReason` -> `func landHaltReason(beam, parentWorktree string, blockers []LandBlocker) string`
- `plan:internal/landingshed#TestFinalize_LandCheck` -> `func TestFinalize_LandCheck(t *testing.T)`

**Edit:**
- `internal/landingshed#Deps`
- `internal/testkit/envkit#nilLegal`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/landingshed#Finalize.Call`
- `internal/landingshed/doc.go`

**Intent:**
D9: pawl's Finalize re-runs `BeamLandRefusal`'s facts under the parent lock and, when they refuse, halts with ways forward it words itself, never `BeamLandRefusal`'s land-it wording, which the standing hold refuses.
It is a backstop for an open member the re-check missed, such as one a stale Bolt on another machine added.

- `Deps` gains `LandBlockers func() ([]LandBlocker, error)`, nil on every recipe but pawl, and `ParentWorktree string`, the beam's parent worktree name its ways forward name.
  `envkit`'s `nilLegal` gains `Landing.LandBlockers`, a documented nil default, so `TestFullEnv_FillsEverySeam` and both wire tests keep passing,
  and `TestLandingDeps_EveryFieldPopulated`'s `unfilled` map gains `LandBlockers` and `ParentWorktree`, which a loom run leaves unset.
  `LandBlocker` holds the blocker's slug and what it is: `Pair` for a pair forked from the beam, else a member with `HasPair` and `Running`.
- `Finalize.Call` runs `landCheck` right after the owner checks, under the lock: a nil seam passes; a read error is a halting Stuck behind `LandHaltPrefix` naming the cause and `lyx loom resume` in the beam worktree; an empty list passes; otherwise a halting Stuck whose reason is `LandHaltPrefix` plus `landHaltReason`.
- `landHaltReason` names, per blocker: for a member with a pair or a run in flight, and for a forked pair, `lyx batten cancel <slug> --remove`, run in the beam's parent worktree, or, when the run is in flight and its pair is not here, on the machine holding the pair, which `lyx batten status <slug>` there shows; for a member, the cancel is followed in the same halt by moving its `project` off the beam with `lyx board upsert`, since the cancel leaves it a `draft` member, which still blocks the landing, as `memberHoldRefusal` (card 18) words it; for a member with no run status, that board move alone; and then `lyx loom resume` in the beam worktree.
  It never names `lyx batten run <slug>`, the landing of a member, which the hold refuses.

`TestFinalize_LandCheck` drives `Call` with the seam: a member with a pair names the cancel in the parent worktree, then the board move; a member in flight with no pair here names the cancel on the machine holding its pair, then the board move; a member with no run status names the board move; a forked pair names its cancel; each ends with `lyx loom resume` and never `lyx batten run`, and none merges or lands; a read error halts naming the resume; an empty list and a nil seam land as before.
`landingshed/doc.go` describes the check and its bound: a member that lands into the beam before Finalize is not caught there, and Worktree-Create's hold read keeps any member from forking once the hold stands.

**ImpactSummary:** one optional seam on Finalize, nil for loom and darn, so their landings are unchanged; the halt goes through the existing halting Stuck path.

**Commit:** 14: Finalize re-checks a beam's members under the parent lock and words its own ways forward
