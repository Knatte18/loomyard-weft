# Card 6 — `parent-review` in the recipe gate vocabulary

**Edit:**
- `internal/shedrecipe#Env`
- `internal/shedrecipe#resolveGateSpec`
- `internal/shedrecipe#resolveGateEntry`
- `internal/shedrecipe#resolveGateClosure`
- `internal/shedrecipe#discussionWriteEntry`
- `internal/shedrecipe/seam_enforcement_test.go`
- `internal/shedrecipe/entries_gate_test.go`
- `internal/shedrecipe/entries_discussionwrite_test.go`

**Prosa:**
- `contracts/specs/shed-recipe-spec.md`

**Uses:**
- `plan:internal/parentreview#GateConfig`
- `plan:internal/parentreview#NewGate`
- `internal/parentreview#Store.BeginRound`
- `internal/shedadapters#NewSingleLLMProducerGated`

**Intent:**
Make `parent-review` a fifth gate name a recipe row can select (decisions D1 and D2), and open a new round on a fresh Discussion-Write spawn (D3).

`Env` gains `ParentReview parentreview.GateConfig`, a whole-struct passthrough like `Env.Landing`, documented as read only by the `parent-review` gate and the DiscussionWrite entry.
`resolveGateClosure` resolves `parent-review` by checking `ParentReview.Store.Root` and `ParentReview.Store.LockDir` with `requireAbsRoot`, the slug with `requireNonEmpty`, both render seams with `requireSeam`, and the two discussion paths with `requireAbsRoot`, then returns `parentreview.NewGate`'s pair; the closure's return widens to carry the optional `Final` closure, nil for the other four names.
`resolveGateEntry` refuses `parent-review` without `pass_on_cap: true` as a construction error naming the row and the key, so a pending result can never reach a must-pass entry.
An unknown name's error lists all five legal values, and `resolveGateSpec`'s doc comment describes the five-value vocabulary.

`discussionWriteEntry` passes a `prepareFreshSpawn` closure when its resolved gate list holds an enabled `parent-review` entry: it calls `ParentReview.Store.BeginRound()` and returns an empty amendment, so a start opens the next round and supersedes an unverdicted earlier one, and an attach continues the latest round.
A disabled entry (`attempts: 0`) or an absent one leaves `prepareFreshSpawn` nil, as today.

Bound: the new name is wired to whichever row lists it, but only as a `pass_on_cap` entry, and the recipe change that lists it on Discussion-Write alone lands in card 12.

The package's import allowlist gains `internal/parentreview`.
`entries_gate_test.go` covers resolution, each missing-Env refusal, the `pass_on_cap` refusal, the five-name error and the `Final` closure being set for `parent-review` only.
`entries_discussionwrite_test.go` asserts `BeginRound` runs on a fresh spawn and not on an attach, and not when the entry is off.
`shed-recipe-spec.md`'s gates paragraph names `parent-review`, the pending state, and the `pass_on_cap` requirement.

**ImpactSummary:** One new Env field and gate name inside shedrecipe; the four existing gate names resolve exactly as before, and only the DiscussionWrite entry gains a fresh-spawn hook.
