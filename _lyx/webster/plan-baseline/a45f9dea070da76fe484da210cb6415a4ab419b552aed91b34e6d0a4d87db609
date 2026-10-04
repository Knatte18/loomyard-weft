# Card 23 — seed-parent-retired

**Edit:**
- `internal/shedrun/seed.go`
- `internal/shedrun/seed_test.go`
- `internal/hubgeom/parent.go`
- `internal/hubgeom/parent_test.go`
- `internal/hubgeom/reedgeom_integration_test.go`
- `internal/loomcli/sharedbootstrap_test.go`
- `internal/battencli/wire_test.go`
- `internal/battencli/arm_seed_test.go`
- `internal/orchcli/smoke_live_test.go`

**Intent:** Retire `shedrun.Seed.Parent` and its read fallback; a run's parent comes only from the pair's origin record (D14).

Confirmed at plan time: no live run carries a top-level seed `parent`.
The five prime run records that do are all in state `done`, and this pair's seed has `parent` only under `params`.

`Seed` loses its `Parent` field.
`decodeSeed` keeps `DisallowUnknownFields` for every other key, but accepts and discards a legacy top-level `parent` key, for example by decoding into an unexported decode-only shape that carries it.
A seed written before the field was dropped therefore still decodes, and nothing re-encodes the key.
The agree/disagree check's comment about the legacy field goes.

`hubgeom.ResolveParent` stops reading the default run's seed.
`decideParent` loses its `legacySeedParent` parameter.
An origin without a `parent_worktree`, or no origin record at all, yields no parent, exactly as a new run without one does today.
The `hubgeom` doc comments describing the seed fallback are rewritten.
If `hubgeom` no longer needs `shedrun` for anything else, the import goes.

Tests:
- `seed_test.go`:
  - a seed JSON carrying `"parent": "ab:hub"` decodes without error and round-trips without the key;
  - an unknown other key is still refused;
  - the old cases that read `got.Parent` are removed or rewritten to that shape.
- `parent_test.go`:
  - the legacy-fallback rows of the decision table become "origin without parent worktree gives none" and "no origin record gives none";
  - a filesystem case writes a seed carrying `parent` beside an origin record naming a parent worktree, and asserts the origin's parent wins and the seed key is ignored.
- `reedgeom_integration_test.go` stops writing `Parent` into its fixture seed, and plants the parent through the origin record instead.
- The batten, loom and orch tests that asserted an empty or set `Parent` drop those assertions.
  The orch smoke test asserts the parent through `hubgeom.ResolveParent` (or drops the seed assertion) and stays compiling under `-tags smoke`.

Grep for `\.Parent` on `shedrun.Seed` values and for `legacySeedParent`; every hit is resolved in this card.

**ImpactSummary:** Removes one Seed field and the hubgeom fallback that read it; old seed files keep decoding; run parentage is origin-record-only.
