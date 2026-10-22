# Fixer report — discussion round 1

Every finding's premise was checked against the code at HEAD and holds.

## Changes

- F1 (BLOCKING), fixed as proposed: D5 no longer has `lenses.Load` log.
  A skipped `remove` entry is reported through a helper configengine exports, which logs at Warn naming the layer file the resolved `remove` came from and each entry, once per process per file and entry list, the way `warnUnknownKeys` dedups.
  Added the Constraints bullet `PATTERN-leaf-packages` and changed the criterion to "logged at Warn once per process".
- F2 (MEDIUM), fixed as proposed: D4 gains a bullet saying a scaffold of a base file for an overlay module, through `Set` or `Edit`, writes its per-name maps empty, as standalone reconcile already writes a fresh file (lenses is not `SeedOnly`, and `mergePerNameMaps` gives an empty map at an absent path).
  Added the criterion that `lyx config lenses --set lens_set=csharp` in a standalone worktree with no `lenses.yaml` resolves the five lenses to the csharp prose.
- F3 (MEDIUM), fixed as proposed: D4's Rationale and the reconcile criterion now say "standalone worktree file", and the Rationale says a hub worktree file is only pruned, with the one-time migration strip judging it against the layers below it.
- F4 (MEDIUM), fixed as proposed: D3's above-base refusal is now for a hub worktree file, and its way forward is to remove `lens_set` from the named file, then set it in the base with `--layer default` from the prime.
  The criterion names the removal.
  The unknown-value refusal keeps its own way forward, now spelled out for a hub and for a standalone worktree, since "the same way forward" no longer fit it.
- F5 (LOW), fixed as proposed: D3 says a local-file `lens_set` keeps `ValidateLocal`'s guard-key refusal, and the criterion says one in a local file is refused as a guard key.
- F6 (LOW), fixed as proposed: the layer criterion now says each layer fan override starts with its base fan's lenses, exactly and in base order, followed only by the layer's additions.

## Self-check

- fix-consistency, LOW: the criterion omits the machine layer D3 named. Fixed differently: D3 now names the local file only and says lenses has no machine layer, following the fix-way-forward finding below.
- fix-consistency, LOW: D6's "the D3 way forward" became ambiguous once D3 has two. Fixed: D6 now spells out the set command.
- fix-consistency, LOW, and fix-proposal, LOW (merged): D4's "scaffold a whole template keep the base template" bullet read against the new scaffold bullet. Fixed: the bullet now excludes an overlay module's per-name maps and points at the next bullet.
- fix-claims, LOW, and fix-proposal side note (merged): the dedup key "per module and entry list" did not match `warnUnknownKeys`, which keys on file path and keys. Fixed: D5 now dedups per file and entry list and names the file, as the review proposed.
- fix-way-forward, MEDIUM: D3 named a machine-file refusal that cannot be reached, since `LayerFile` gives the machine layer a file only for a hub-wide module (`internal/configengine/layers.go:117-120`). Fixed: D3 names the local file only.

## Disputed

None.

## Deferred

None.
