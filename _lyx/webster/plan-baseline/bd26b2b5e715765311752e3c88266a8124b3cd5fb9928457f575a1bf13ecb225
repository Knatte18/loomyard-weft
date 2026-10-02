# Card 9 — seed-parent-retired

**Edit:**
- `internal/battencli/arm.go`
- `internal/battencli/wire.go`
- `internal/loomcli/sharedbootstrap.go`
- `internal/shedrun/seed.go`
- `internal/shedrun/doc.go`
- `internal/battencli/arm_seed_test.go`
- `internal/battencli/wire_test.go`
- `internal/battencli/lifecycle_integration_test.go`
- `internal/loomcli/sharedbootstrap_test.go`

**Prosa:**
- `CONSTRAINTS.md`
- `CLAUDE.md`

**Intent:**
Stop every write of `shedrun.Seed.Parent`, removing #341's cause: no seed records a parent and nothing reads the caller's `LYX_STRAND_NAME` on the write side (Decision 2).

- `internal/battencli/arm.go`: the batten seed drops `Parent: os.Getenv(agentname.StrandNameEnv)`; drop the imports that leaves unused.
- `internal/battencli/wire.go`: `childSeedFor` loses its batten-seed argument and builds `Seed{Recipe, Driver, Params}`; Seed-Child's `WriteSeed` closure no longer reads batten's own seed; update the two comments that say the child inherits batten's parent.
- `internal/loomcli/sharedbootstrap.go`: `loomSeedFor` drops its `caller` parameter and the seed's `Parent`; the call site drops `os.Getenv(agentname.StrandNameEnv)` and any import left unused.
- `internal/shedrun/seed.go`: `Seed.Parent` stays in the struct so a seed written before this change decodes under `DisallowUnknownFields`; its doc says no code path writes it, and it is read only by `internal/hubgeom`'s parent resolver as the legacy fallback. Adjust `WriteSeed`'s doc to match.
- `internal/shedrun/doc.go`: replace the paragraph on the write-once parent with that legacy-only description, pointing at the Agent Name Invariant.
- `CONSTRAINTS.md`, Agent Name Invariant: add a bullet that a run's parent is the worktree its pair was created from, recorded once in fabric's origin record; its agent name is resolved at use as the orch role's name in that worktree through `internal/agentname`, and is never stored in a seed or read from the caller's environment.
- `CLAUDE.md`: delete the TEMPORARY item for #341.

Tests: `arm_seed_test.go` asserts a seed armed with `LYX_STRAND_NAME` set records no parent; `wire_test.go` replaces `TestChildSeedFor_CarriesTheBattenSeedsParent` with an assertion that the child seed carries no parent; `sharedbootstrap_test.go` follows `loomSeedFor`'s new signature and asserts no parent.
In `lifecycle_integration_test.go`, `TestBattenIntegration_SeedChild_WritesASeedTheChildBootstrapAgreesWith` also asserts the child seed's `Parent` is empty and the pair's origin record names the prime as `ParentWorktree`, which with card 2 is what gives the pair's strands the prime's orch name as `LYX_PARENT`.
`internal/shedrun/seed_test.go`'s existing `TestSeed_ParentRoundTripsAndIsOmittedWhenEmpty` already proves a seed carrying `parent` still decodes, so it stays as it is.

**ImpactSummary:** Removes the three seed-parent writes and the caller-environment read; readers fall back to the origin-record resolver, and pre-existing seeds still decode.
