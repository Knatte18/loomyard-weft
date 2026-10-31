```yaml
lens: board-scope
outcome: reported
```

Task: check the decision record against the board — the entry's goal points, touching entries, peer runs, parent commits and decision notes.

**Findings**

1. LOW — D12 / Out-of-scope line about `profiles.yaml`. D12 relies on `KeyMoves` to carry tuned `plan_*` keys into `tapestry.yaml`. The entry says the orch rewrites the hub's `profiles.yaml` after landing and the run writes no migration for it. If `KeyMoves` also rewrites profile blocks, through run settings or profile resolution, then the run and the orch would both migrate `profiles.yaml`. Evidence: entry "Config split" paragraph. `internal/configreg/runsettings.go` rewrites run settings through `KeyMoves`, and the record does not say whether profile blocks go through it. Fix: state in D12 that `KeyMoves` reaches base files and run settings only, never `profiles.yaml`. If it does reach `profiles.yaml`, say so and drop the out-of-scope claim.

2. LOW, already raised under Open risks, not a new `decision` finding — D13 and `batten.yaml`. The parent said "in `shed.yaml` alone". The record keeps `batten.yaml`'s overrides and flags this for the parent, which is the right handling. No further action.

**Goal points of `run-verbs-shared`**
- Three layers, loom as a contract, checked by Shed at load: met (D1).
- `lym loom` verbs act on loom runs only, pawl left alone: met (D2, with the parent decision on pawl).
- Producers stay in their owning modules, and a variant owns only producers no other recipe uses: met (D4, D8).
- Record with Discussion or written mechanically, and `decision add` appends to it: met (D4, D5, D6).
- Code-Review measures against the record, not the entry snapshot: met (D7).
- Shared code written once (Describe, description commit, writer wrappers, gate and validator, verify source, routing, shed and retired lists): met (D5, D8).
- One row order with Drift-Verify before Code-Review: met (D9).
- Verbs that apply to one recipe (peers, `--pattern-*`, `--amends-plan`, the plan verbs): met (D3).
- Config split between `loom.yaml`, `tapestry.yaml` and `darn.yaml`: met (D12).
- Tapestry-Preflight goes and the `batcher.yaml` load moves: met (D10).
- `legacyRecipeBeam` goes: met (D11).
- The four Open items: arbiter keys settled by D13, the empty-section question by D5, the outdated-entry check by D6, and `lym loom` versus `lym shed` by D14. All met.
- The brief's Hardener variant: the dated body names only tapestry and darn as loom recipes, so excluding Hardener does not conflict.

**Entries, peers and commits checked**
- `pawl-loom-recipe`, which depends on this task: no conflict. D1's writer-engine set will need pawl's member-run writer added, which is that task's job, and D2's exception is the hand-off point.
- `seed-starts-at-start`: no conflict. Preflight stays the entry row of both loom recipes.
- `finalize-fix-in-place`: no conflict. It is out of scope and named under Open risks.
- `hardener`: no conflict. It is parked and out of scope.
- `beam-promote-keeps-discussion`: no conflict.
- `loom-beam-darns`: no conflict. A mechanical darn record fits pieces cut by Discussion.
- Peers `board-push-race` and `hold-notice-operator-turn`, both at Darn-Write: their scope is unrelated, so no conflict.
- Parent commits since the fork: none listed.

**Decision notes**
- No `Decision for run-verbs-shared:` note on the board. The earlier board list showed none, and none was unreachable.
- The parent decisions arrived as creel mail (#156 and #159) and are recorded as D2, D13 and D14.

**Tool calls**
- `lym loom peers`: peer runs and parent commits, which my context did not have.
- `lym board get loom-beam-darns`: the full note body, which my context held only as a title line.
