```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: I checked each claim the fix added or rewrote against the code at HEAD.

**MEDIUM.** Decision record D5, the `lym loom start` bullet, which now says "the check runs when bootstrap finds no status file, before `loomshed.Seed` writes it".
- **Summary:** bootstrap never looks for the status file. It learns whether one exists only from `loomshed.Seed` returning `ErrSeedExists`. So the placement described needs a new stat-then-seed probe, which the code's own comment rules out.
- **Evidence:** `internal/loomcli/sharedbootstrap.go:120-125`: "A stat-then-seed probe would reintroduce the race that the seeder's single lock closes. ErrSeedExists is therefore the only accepted outcome." The decision should say how a first start is told apart without that probe. One option is to judge the check inside the seeding lock. If it runs after Seed succeeds, it must say what happens to the status file Seed just wrote.

**LOW.** D2 rationale, "Preflight's batcher and drift checks guard config only the build rows read; its worktree and fabric checks also cover Discussion's commit". This wording came from the reviewer's proposal.
- **Summary:** the list of what Preflight runs leaves out its stencils check and its hardener check. "Config only the build rows read" covers only the checks it names, so a reader could take the list as complete.
- **Evidence:** `internal/preflightshed/preflight.go:107-117` (`stencilsReasonPrefix`, `hardenReasonPrefix`) and `:66-68` (`Checks` holds optional checks beyond batcher and drift).

These claims hold:
- **D3:** the skip falls back to the block's entry (`producer.go:130-134`).
- **D4:** `EngineRow` reads expanded rows, because `parseEmbeddedRecipe` calls `shedbuild.Expand` (`names.go:156`).
- **D5:** `startClearsMotifRun` admits raddle starts only, as the record says.
- **D7:** the reinterpretations are registered in `internal/configreg/configreg.go:171`, next to the arras one.
- **D9:** `runUsageRecord` sets `stored_record` from `entry.Discussed` alone.
- **D2:** motif has no Preflight row (`contracts/recipes/motif-recipe.yaml`).

Tool calls:
1. One Bash call read `parseEmbeddedRecipe`, the row names in `motif-recipe.yaml` and the check list in `preflightshed/preflight.go`. My context had never read whether `EngineRow` sees expanded rows, whether motif holds a Preflight row, or the full set of Preflight checks.
