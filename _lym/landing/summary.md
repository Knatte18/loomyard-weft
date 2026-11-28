# Add jacquard blocks to run profiles, carried over from tapestry values

Every run profile that holds a `tapestry` block now also holds a `jacquard` block, and `defaults.jacquard` names a profile.
Jacquard is the recipe for tapestry's rows after Discussion.
This change adds only the profile blocks, ahead of that recipe.
It adds no `jacquard` recipe, and raddle's check still accepts only `tapestry` and `darn` as a part's recipe.

- `internal/runprofile/template.yaml`: `speed`, `light`, `solid` and `wide` gain a `jacquard` block,
  equal to their `tapestry` block without `loom.discussion`.
  `defaults` gains `jacquard: solid`.
- A one-time reinterpretation, `profiles-jacquard-blocks`, carries a hub's tuned tapestry values into its jacquard blocks.
  It is registered after the arras one in `internal/configreg`.

## Design decisions

**Why a reinterpretation is needed.**
The reconcile's template follow alone would give a shipped profile the template's jacquard values,
whatever the hub tuned under its `tapestry` block.

**What the rewrite does.**

- Every profile holding a `tapestry` block, operator-added ones included, gets a `jacquard` block derived from it.
  A differing `jacquard` block is replaced.
  A profile with no `tapestry` block is left as it is.
- `defaults.jacquard` is set to the profile `defaults.tapestry` names when that profile holds a `jacquard` block,
  and is dropped otherwise.
- A derived block that differs from `profiles.template-base` counts as the hub's own on later reconciles.
- It runs once. A profile added afterwards gets its `jacquard` block from the operator.

**What it leaves to a hand edit.**
Each of these is reported as a flagged line with a way forward:

- a profile or default supplied through a YAML merge key, where it would have to change;
- an existing `jacquard` block or default that would change and carries a YAML anchor.

The reinterpretation is marked as applied even when it flags something, so a later reconcile does not retry it.

**A related fix to the arras rewrite.**
`ReinterpretArrasDefault` now flags a `defaults.arras` held both as an own key and through a merge key.
Removing the own key alone would have left the merged default in force.

## To check by hand

- The added rewrite tests run on file contents held as strings; none of them drives it through the reconcile walk.
  After the next reconcile of a real hub, check `profiles.yaml`: tuned values under `jacquard`, `defaults.jacquard`,
  and any flagged lines.
- A rewritten file is re-encoded with two-space indent.
  The rewrite keeps comments, but check the diff for other layout changes.

## Merge danger

**Door:** one-way
**Blast radius:** profiles
