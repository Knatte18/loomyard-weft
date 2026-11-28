# Seed no arras run profile blocks, and mark arras as kept but not offered

The arras recipe is no longer offered for new work, yet the run profile template still seeded an `arras` default and an `arras` block in each starting profile.
`lym board profiles` therefore listed arras settings, and the overview and glossary described arras alongside the recipes that are offered.

## What changes

- `internal/runprofile/template.yaml` drops `defaults.arras` and the `arras` block of `speed`, `light`, `solid` and `wide`.
  The hub reconcile drops these keys from an existing `profiles.yaml` wherever the file still holds the template's earlier value;
  an arras key the operator changed stays, as for any other template change.
- An arras run naming no profile now runs on the hub's own config, the rule for any recipe with no default.
  The recipe itself, its split perch and its `loom.split.*` config keys stay in code.
- `TestTemplateLadder` now pins that no starting profile holds an arras block and that arras has no default.
  `TestLoad` and the board `profiles` verb test drop arras from the expected defaults.
- `docs/overview.md` marks arras as kept in code but not offered wherever it lists the recipes, and states that no starting profile covers it.
  The glossary's arras row says the same, and `internal/runprofile/doc.go` records the rule.

## Design decisions

- The `configsync` fixture `testdata/profiles-flat.yaml` keeps its arras blocks:
  it is the template as it stood before the flat-key move, and the key-move test reconciles it into the current template,
  so that test now also carries the arras keys' drop.

## Merge danger

**Door:** two-way
**Blast radius:** runprofile
