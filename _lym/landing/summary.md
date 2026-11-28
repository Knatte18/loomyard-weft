# Drop arras from the starting run profiles and mark it as not offered

The `arras` recipe is kept in code but no longer offered, yet the starting run profiles still covered it.
`lym board profiles` therefore listed it, and `docs/overview.md` described it like any offered recipe.
The weaving glossary already said no entry may be given it, but not that no starting run profile should cover it.

## What changes

- `internal/runprofile/template.yaml` holds no `defaults.arras` and no `arras` block in any profile,
  so `lym board profiles` on the starting profiles no longer lists arras.
- The overview, the weaving glossary and the `internal/runprofile` package doc say arras is kept in code but not offered.
  The recipe itself is not touched.
- An existing hub's `profiles.yaml` follows the template where a template base is kept beside it:
  an arras block or default left as shipped is removed, and an edited one stays, with the one exception below.
  Without a template base, the arras blocks and default are kept and reported as differing, so arras is still listed.
- A new one-time reinterpretation, `profiles-arras-default-dropped`, handles the exception:
  an edited default naming a profile left with no arras block would keep the file from loading, so it is dropped.

## Design decisions

- **No default for arras.** An arras run naming no profile runs on the hub's own config.
  A profile with no arras block does not cover the recipe, so asking it for arras settings is refused.
- **Why the reinterpretation exists.** A hub that repointed `defaults.arras` at another starting profile keeps that edit,
  while the profile's untouched arras block is removed.
  The default then names a profile with no arras block, and the file does not load.
- **It drops the default and flags it.** The notice says how to get a default back: add the block, then set the default again.
  A default naming a profile that still has an arras block is left alone.
  Comments and all other keys are kept, though the file is re-encoded with two-space indentation.
- **Aliases and merge keys.** A broken default written through one is not rewritten.
  It is flagged for a hand edit, and the file stays unloadable until that edit is made.

## To check by hand

- No test feeds `ReinterpretArrasDefault` (`internal/runprofile/reinterpret.go`)
  a default written through an alias or a merge key, so check that branch by reading it.

## Merge danger

**Door:** one-way
**Blast radius:** runprofile

The door is one-way because the reconcile rewrites a hub's `profiles.yaml`, and reverting the commit does not undo that rewrite.
