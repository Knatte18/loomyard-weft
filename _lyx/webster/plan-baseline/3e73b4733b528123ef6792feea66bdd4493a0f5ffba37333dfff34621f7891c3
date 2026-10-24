# Card 10 — the new keys in the tier fingerprint, and carry: false

**Edit:**
- `internal/gatecli#tierFingerprint` -> `func tierFingerprint(tier gateslot.Tier, build gateslot.BuildStep, inert []string) string`
- `internal/gatecli#greenRun`
- `internal/gatecli#openGreen` -> `func openGreen(gateDir, root, goBinary string, cfg gateslot.Config) *greenRun`
- `internal/gatecli#greenRun.subtract`
- `internal/gatecli#greenRun.recordPass`
- `internal/gatecli#diffBaseGroups`
- `internal/gatecli#runTest`
- `internal/gatecli#scopeInvocations`
- `internal/gatecli#TestGreenRecord`
- `internal/gatecli#TestGreenRecord_NewestWholeAncestor`
- `internal/gatecli#TestGreenRunRecordPass`
- `internal/gatecli#TestPlanTierRuns`
- `internal/gatecli/doc.go`

**Uses:**
- `internal/gatecli#greenTier`

**Intent:**
Make every new key that changes what a tier tests void its green entries, and add the tier that never carries (Decisions 5, 6 and 8).

- `tierFingerprint` hashes, beside today's fields, the tier's `Filter`, `SharedBuild` and `NoCarry`, the `build` declaration when the tier sets `SharedBuild`, and `inert` when it is non-empty, each only at a non-default value, through `omitempty` JSON fields added to the hashed struct after today's.
  So an existing tier, with none of the keys set and the hub's `inert_paths` empty, keeps its fingerprint and its green entries, and a changed `filter`, `shared_build`, `carry`, opted-in `build` or `inert_paths` list stops it from carrying.
- `greenRun` gains `build` and `inert`, which `openGreen` takes from the config it is now told, with its `scoperGo` read from `cfg.Scoper`; `runTest` passes `cfg`.
  `subtract`, `recordPass` and `diffBaseGroups` fingerprint through them, so a base and a carry are looked up under the same fingerprint a pass writes.
- A tier with `NoCarry` is never carried: `subtract` keeps every invocation of it, and `recordPass` writes no entry for it, so a timing or capture tier on an unchanged tree runs again.
- `scopeInvocations` runs a `NoCarry` tier whole, over its `all_targets` in its `dir`, under every scope form but explicit targets: a computed, `--base` or `--card` scope, a narrowed `go` or `dotnet` share, the empty scope and a `noTests` share alike, so no computed, inert-only or no-class scope leaves it unrun.
  Explicit targets narrow it as they narrow any tier, and a listed tier whose `covers` names it still drops it, which is the Bound of Decision 8 that `docs/dotnet-hub.md` tells a hub to keep it out of.
- `doc.go`'s Green record sentence on what `tierFingerprint` hashes and its empty-scope sentence are rewritten for the new fields and for a `carry: false` tier running under an empty scope, and the section names `carry: false` with its Bound: a verify site's verified-tree record, explicit targets and `covers` still skip or narrow such a tier.

Tests:
`TestGreenRecord`'s fingerprint row asserts that `tierFingerprint(greenTier(), gateslot.BuildStep{}, nil)` equals the hex string today's code returns for `greenTier()`, captured before the change, and that a changed `Filter`, `SharedBuild`, `NoCarry`, a changed build under `SharedBuild` and a non-empty inert list each change it, while a changed build without `SharedBuild` does not.
`TestGreenRecord`'s subtract table gains a row: a `NoCarry` tier with a whole entry for the tree keeps its invocation.
`TestGreenRecord_NewestWholeAncestor` fingerprints through the new signature.
`TestGreenRunRecordPass` gains a row: a `NoCarry` tier's pass writes no entry.
`TestPlanTierRuns` gains rows: a `NoCarry` tier runs whole under a narrowed computed scope, a `--card` scope, the empty scope, a `noTests` share and a narrowed `dotnet` share, in the last with no terms and `partial` unset, and over the explicit targets of an explicit-targets call.

**ImpactSummary:** `tierFingerprint`'s three production callers and `openGreen`'s one, `runTest`, change on this card; hubs that set none of the new keys keep every fingerprint, so no green record is voided by the upgrade.
