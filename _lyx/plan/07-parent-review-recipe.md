# Card 7 — parent-review-recipe

**Edit:**
- `internal/shedrecipe/entries_gate.go`
- `internal/shedrecipe/entries_discussionwrite.go`
- `internal/shedrecipe/entries_gate_test.go`
- `internal/shedrecipe/entries_discussionwrite_test.go`
- `contracts/recipes/loom-recipe.yaml`
- `internal/loomcli/parentreview_wiring_test.go`

**Prosa:**
- `contracts/specs/shed-recipe-spec.md`

**Uses:**
- `plan:internal/parentreview#Store.PrepareRound`

**Intent:**
Wire the multi-round, capped parent review into Discussion-Write's row.

In `resolveGateEntry`/`resolveParentReviewClosure`, the `parent-review` entry:

- sets `GateEntry.MayHold` (card 3) itself; no recipe key and no other entry name can set it;
- tells its `attempts` value to the gate as `GateConfig.Cap`, so `attempts: 3` means three rejected rounds;
- is refused when its element sets `pass_on_cap: true`, with an error naming the row, the key and that the entry escalates at its cap rather than letting the run through.

The existing "a `Final` closure requires `pass_on_cap: true`" refusal is replaced by that one, since `parent-review` is the only entry with a `Final`.
`discussionWriteEntry`'s fresh-spawn preparation calls `Store.PrepareRound` instead of `BeginRound`, so a re-spawn after a reject or a superseding approve keeps the latest round for the gate to read.
Update `resolveGateSpec`'s and `discussionWriteEntry`'s doc comments to match.

In `contracts/recipes/loom-recipe.yaml`, Discussion-Write's `parent-review` gate becomes `attempts: 3` with no `pass_on_cap`, and its comment says the parent reviews every rewrite after a reject, the third reject halts the run `blocked`, and `lyx loom review approve` lifts it.
In `contracts/specs/shed-recipe-spec.md`, rewrite the `parent-review` paragraph: the entry is must-pass and may hold the run, its `attempts` is the reject cap, `pass_on_cap` is refused on it, and a fresh spawn keeps the latest round after a reject or a superseding approve.

Tests: in `entries_gate_test.go`, a `parent-review` element without `pass_on_cap` builds an entry with `MayHold` set and the gate's cap equal to `attempts` (observable through the built closure over a temp store), and one with `pass_on_cap: true` is refused naming the key.
In `entries_discussionwrite_test.go`, the preparation keeps a reject round instead of beginning a new one.
In `internal/loomcli/parentreview_wiring_test.go`, `TestDiscussionWriteRow_GateListIsDiscussionThenParentReview` pins `attempts: 3` and no `pass_on_cap` on the embedded recipe.

**ImpactSummary:** Changes how the recipe's parent-review entry is built and configured; other gate names resolve unchanged.
