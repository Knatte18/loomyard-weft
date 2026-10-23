---
round: 3
focus:
  - "Card 19 (`_lyx/plan/19-arbiter-amendment-target.md`): the arbiter prompt's `amendment_target` must name `Plan-Burler-Review` for a stop after plan approval even when the drive started before approval. The Go driver builds its `arbiterDeps` once per drive (`internal/loomcli/drivewiring.go`, `goDriverDepsFor` calling `arbiterDepsFor`) and reuses them for every stop, while `arbiterCall` copies `c.deps.Prompt` at each spawn (`internal/loomcli/arbiter.go`). Settle whether the card reads the approval at each spawn rather than at wiring time, whether its Edit targets list every symbol its Intent changes, whether its Intent states the signature of each changed symbol compilably, and whether its tests include a row that builds before approval and spawns after it."
  - "Card 24 (`_lyx/plan/24-planparser-target-holds.md`): `TargetHolds` compares a quarry-minted member glyph string against a plan ref's canonical glyph string. Settle whether the card makes the implementer read both through `glyph.Parse` and the parsed unit, owner chain and name, as `tools/plandag`'s `columnsOverlap` does, and never by string prefix, per PATTERN-glyph-conversion-chokepoint."
  - "Cards that consume the changed symbols: settle whether any later card still reads `arbiterDeps.Prompt.AmendmentTarget` as a wiring-time value, or calls `arbiterDepsFor` expecting the old shape."
---

Round 3 re-checks the text round 2's fixes touched, plus the cards that depend on it.
