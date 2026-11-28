# Run profiles gain jacquard blocks carried from their tapestry blocks

A jacquard run reads its recipe's block from the run profile it takes, and a profile holding no jacquard block refuses it.
This change gives the profiles those blocks, in the template and in every existing hub's `profiles.yaml`.

## What changes

- `internal/runprofile/template.yaml`: the `speed`, `light`, `solid` and `wide` profiles each gain a `jacquard` block,
  their `tapestry` block without `loom.discussion`, since a jacquard run has no Discussion.
  `defaults` gains `jacquard: solid`, as tapestry's.
- `internal/runprofile/reinterpret.go`: a one-time reinterpretation, `ReinterpretJacquardBlocks` under the ID `profiles-jacquard-blocks`.
  The reconcile's template follow alone would give a shipped profile the template's jacquard values whatever the hub tuned in its tapestry block.
  The reinterpretation runs after that follow and:
  - gives every profile holding a tapestry block a jacquard block equal to that block minus `loom.discussion`,
    dropping the `loom` module when Discussion was its only area;
    a profile left as shipped already matches, so the file comes back byte for byte;
  - sets `defaults.jacquard` to the profile `defaults.tapestry` names when that profile then holds a jacquard block,
    and drops `defaults.jacquard` otherwise.
  It keeps comments, reports one line per changed block or default with a way forward,
  writes an aliased profile's block into the mapping the alias names,
  and flags a profile or default supplied through a merge key for a hand edit.
  The arras rewrite's encoder moves into a shared `encodeDocument`.
- `internal/configreg/configreg.go`: the profiles module registers the new reinterpretation after the arras one.
- `internal/runprofile/doc.go` describes the jacquard blocks and the carry.

Once written, a derived block that differs from `profiles.template-base` is kept as the hub's own on later reconciles,
since the follow keeps a unit the file holds otherwise than its base.

## Design decisions

- The block key is the recipe name `jacquard`, held as the unexported constant `jacquardRecipe` in `internal/runprofile`.
  `internal/shedrun` declares no jacquard recipe name on this branch, and the profile block key needs no other part.
  Once shedrun declares one, the constant can give way to it.
- A jacquard block already present and equal to the derived one is left untouched, so the template's flow-style blocks keep their form.
  A block the rewrite writes is encoded in block style, which quotes every scalar safely.

## Tests

- `TestReinterpretJacquardBlocks` covers a tuned shipped profile, an operator-added profile with comments kept,
  a changed tapestry default, a jacquard default dropped when tapestry has none, an aliased profile,
  and a profile supplied through a merge key.
- `TestReinterpretJacquardBlocks_ShippedTemplateUnchanged` pins that the template is a fixed point of the rewrite.
- `TestTemplateLadder` and `TestLoad` pin each starting profile's jacquard rounds and the jacquard default;
  `TestLookup_ProfilesCarriesTheJacquardBlocksReinterpretationAfterTheArrasOne` pins the registration order.
- The board CLI's profiles step expects the new default.

## Merge danger

**Door:** one-way
**Blast radius:** profiles
