```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked the record's claims, bounds and guarantees on the block design.

**Findings**

1. MEDIUM — D2 (expansion refusals). The bound "an inlined row's edges stay inside its own recipe" covers only `on_stuck` and `on_reroute`. It does not cover a non-empty `on_done`.
   Counterexample: a `cartoon` row with `on_done: Drift-Baseline` parses cleanly, because `shedbuild.Parse` validates no routing. After expansion, inside `arras`, that name resolves to the parent's row. The run then leaves the block by an edge that is not the block's exit, and nothing refuses it. A typo naming a row in neither recipe is caught only later, by shedengine's validate, as a dangling target.
   Fix: add "an inlined row whose non-empty `on_done` names a row outside its own recipe" to D2's refusals and to the acceptance bullet.

2. MEDIUM — D8 rationale and acceptance. The record says the operator-owned profiles file "would otherwise refuse an `arras` task". That is only half true.
   `internal/battencli/runsettings.go` `prepareRunSettings` refuses only a *named* profile that lacks a block for the recipe. With no profile named, `profiles.Default("arras")` returns `""` and "the task's settings stand alone".
   So without `profile_recipe`, an unprofiled `arras` task is not refused. It runs silently on the plain `loom.yaml` and `tapestry.yaml` budgets instead of tapestry's default `solid` profile. For the bench that is worse than a refusal.
   The decision stands; the rationale should name the silent difference. The plan writer should also know of a second reader: `internal/configreg/runsettings.go` `checkProfiles` walks profile blocks by recipe name. D8's "every reader resolves through one accessor" has to cover both readers. `configreg` would then import `loomrecipe`, which should be checked against the leaf and config-layer rules.

3. LOW — D6 (goto entries carry the block path). Goto appends its history entry from `GotoRequest.Target` (`internal/shedengine/goto.go:136`, and the relaunch path at `:250`). That request's `Producers` come from the routing projection, not from the built Shed.
   The guarantee therefore holds only if the projection D11 adds also carries each row's block path. D11 does not say it does. The same gap applies to `loomrecipe.Rows` and `RecipeRow`, which `lym loom status`'s row list reads.
   Fix: state that the routing projection and the row accessor both carry the block path.

4. LOW — D10 ("accepted and wired by name the same way"). `lym loom peers` reports each peer's `row` as a 1-based index into that peer's own recipe (`internal/loomcli/peers.go` `recipeRow`).
   An `arras` peer at Discussion-Write reports row 3, while a tapestry peer at the same row reports row 2. Every later row is likewise off by one, so peer positions compared across the two recipes misalign.
   Fix: name it in Open risks, or note that peers compare row names rather than indexes.

5. LOW — D10 acceptance ("expanded rows equal tapestry's rows plus `Cartoon`, edge for edge"). As written this cannot hold: Preflight's `on_done` is `Cartoon` in `arras` and `Discussion-Write` in tapestry.
   Fix: say "edge for edge, bar Preflight's `on_done`", so the test does not pin an impossible equality or get loosened ad hoc.

**Claims attacked**

| Decision | Claim | Result |
|---|---|---|
| D1 | The block row's closed key set is refused otherwise | held |
| D2 | Rows keep their names, so goto, pause and the friction path are unchanged | held |
| D2 | The row-name check refuses the same recipe used twice | held |
| D2 | Segment collision is refused | held |
| D2 | Self-inclusion is refused | held |
| D2 | Inlined edges stay inside the block | broken (finding 1) |
| D3 | A goto onto the block row bypasses its gate (EnteredRow is written by Goto) | held |
| D3 | A pass-over records no inlined row | held; the skip branch must test the skip target, not `OnDone == ""` (plan detail) |
| D3 | The skip-target bound: only a block row carries one | held |
| D4 | A Stuck with no `on_stuck` halts at the inlined row; resume re-calls it | held |
| D5 | An unknown check is refused at build | held; the status path never builds, which is acceptable |
| D6 | Every entry carries the block, goto entries included | broken for goto unless the projection carries it (finding 3) |
| D6 | The additive field is tolerant | held; an older binary cannot step an `arras` run, so it never drops the field |
| D7 | Progress follows the rewritten `on_done`; budgets are keyed by segment | held |
| D8 | An existing hub would refuse an `arras` task | broken for the unprofiled case (finding 2) |
| D9 | `cartoon` cannot be seeded (batten checks the recipe against `shedrun.RecipeNames`) | held; a hand seed is refused by the loom contract check |
| D10 | `arras` is treated like tapestry everywhere | broken for peers' row index (finding 4) |
| D10 | The edge-for-edge pin | wording broken (finding 5) |
| D11 | One expansion point | held, provided the projection and row accessor are extended (finding 3) |

**Tool calls**

- Grepped the `runprofile` callers: the context did not show who reads profile blocks by recipe name.
- Read `peers.go` around `recipeRow` and `peerRecipe`: the context did not show how peers compute row positions.
- Grepped recipe validation and the history-append sites: the context lacked where goto and reroute append entries and where a recipe name is validated.
- Read `configreg/runsettings.go` `checkProfiles` and `battencli/runsettings.go` `prepareRunSettings`: the context lacked what happens to a recipe with no profile block.
