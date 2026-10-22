---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/yamlengine/set_test.go:308
    summary: Refusal paths of a per-name entry key P.<name>.<rest> have no test row
    fix: Add a row to TestSetValues_Refusals where existing profiles is a sequence and the pair is profiles.costed.budget, and a wantErr row to TestSetSparse where existing profiles.costed is a scalar and the pair is profiles.costed.budget; each expects an error naming the path and shape.
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: consistency
    location: internal/configsync/configsync.go:40
    summary: formerTemplateValues comment does not say a per-name map is compared per unit, which is why the batcher entry holds one key
    fix: In the comment above formerTemplateValues, say that under any mapping other than an open map, a per-name map included, each leaf unit is compared alone.
    origin: lens:code-pattern-fit
  - id: F3
    severity: LOW
    class: design
    location: internal/batcher/config_test.go
    summary: profileOrigins error for a profiles key that is no mapping has no test row
    fix: Add a TestActive error row seeding a layer file with profiles as a list, expecting an error naming batcher.yaml and marked invalid.
    origin: lens:code-tests
---

The change makes the amended board entry's two asks (#833, #834) and stays inside its scope; the focus questions settle as follows.

- `SetValues`/`SetSparse` handle `P.<name>.<rest>` for names the template holds and lacks; unknown rest and shape mismatch are refused.
- `ValidateLocal`/`ValidateMachine` leave lenses and gate closed, because their guard category still refuses.
- The commented-empty-mapping fixes each have a regression row.
- The retired-key refusal names the highest layer holding the profile.
- The registry fingerprint hashes `PerNameMaps`, so the summary's reconcile claim holds.
- The three `doc.go` files match the new behavior; no new cross-cutting invariant needs a PATTERN entry.

### [LOW] Entry-key refusals untested

**Location:** internal/yamlengine/set_test.go:308
**Issue:** The `perNameShapeError` and `mappingAt` failures on the entry-leaf branches have no row; existing refusal rows use `names.x`, not a mapping-entry key.

### [LOW] formerTemplateValues comment

**Location:** internal/configsync/configsync.go:40
**Issue:** The comment names open maps and "any other mapping" but not per-name maps, so the narrowed batcher entry is unexplained.

### [LOW] profileOrigins error path untested

**Location:** internal/batcher/config.go (`profileOrigins`)
**Issue:** The error return for a non-mapping `profiles:` is reached by no test.

## Rejected

- SetValues keeps a whole profile on a one-key set (lens:code-correctness, MEDIUM): `SetValues` serves only a full, non-sparse base file, where holding the whole profile is the file's shape; a hub layer goes through `SetSparse`, which writes only the leaf.
- `FillMissing` clears flow style on every append (lens:code-correctness): cosmetic, valid YAML, and the same code path the focus asks to keep valid.
- `SetSparse` lacks `liftEmptyValueComments` (lens:code-affected-set): unconfirmed, and `SetSparse` resets style through `mappingAt`, giving valid output.
- "each name one unit carried whole" in yamlengine `doc.go` and `reconcile.go` is stale (two lenses): still true for reconcile and `--set`, and the added sentence states the merge and fill behavior.
- Empty-sequence branch of `liftEmptyValueComments` has no row (lens:code-tests): a defensive branch of the shared helper, not a behavior the change adds for a caller.
- `SetSparse` has no list-leaf row (lens:code-tests, NIT): the shared `setPerNameEntryLeaf` list branch is pinned through `SetValues`.
