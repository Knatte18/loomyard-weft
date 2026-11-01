```yaml
lens: code-affected-set
outcome: reported
```

**Lens code-affected-set: one task, list per change below.**

**Per change, the set I found:**
- **`SamplerSession` engine and `RecipeSampler` name**
  - Consumers I checked: the registry, `LoomBuildEngines` and `loomrecipe.RecipeEngines`, `RowNames`, `Rows` and `embeddedRecipeSource`. Also `shedrun.RecipeNames`, `shedcli/table.go`, battencli `childRecipeSupported` and its refusal text, and the board's `normalizeTask` message. All are updated in the diff. Covered.
  - `loomrecipe.IsFinalizeRow` and `BouncerRunSubdir` iterate only the tapestry and darn recipes. Sampler's `Finalize` and `Code-*` rows share names, engines and `run_subdir` with those recipes, so both still answer correctly. Not a defect.
  - `reportengine/runs.go:24` (`runRecipes`) and `timingspans.go:82` list only tapestry and darn. Nothing in the decision record asks for sampler reports, so this is not a finding.
- **`wireDarn` split into `wireTreeScope` and `wireDarn`**
  - `SegmentBounces` is no longer set in `wireDarn`. The grep shows it set for every recipe at `wiring.go:813`, so nothing is lost.
  - The `darnDeps`, `wireDarn` and `darnTreeScope` callers are consistent with the new `recipe` parameter. Covered.
- **`rerouteReviewed` and `codeBudgetZero` now take `recipe`**: the `landingDeps` callers are both updated, and `CodePerchGate` reads the row gate. Covered.
- **`agentActivityFor` now takes `runID`**: the one caller, `arm.go:432`, is updated. Covered.
- **`childAtWaitingRow` and `agentActivityFor` read `loomshed.WaitsOnOperator`**: no private two-row copy is left in `internal/battencli` or `internal/loomcli`. The unused `pawlengine` import is removed. Covered.
- **`decision add` caller rule**: `samplerStrand` is wired in `preRun`. The package doc, the `Long` text and the refusal way-forward constant are updated. Covered.
- **`pattern/PATTERN-daemon-wakeups.md`**: the commit `099f1f98` adds the Sampler-Session wait. The diff declares no other PATTERN entry change, and no `Pattern-Departure:` trailer appears in the range. Covered.
- **Run-profile template**
  - Each ladder profile gains a `sampler` block and there is no `defaults` entry, as D2 asks.
  - `runprofile/doc.go:12` says "sampler has none", and the doc is updated. Covered.
- **Stencil**: it carries the parent and edit directives, the decision record path, the slug, `decision add --by operator`, `sampler-done`, notes-only board writes, no dev build in this hub, and `tmux -L` with the slug. It names no denied form. Covered.

No findings from this lens.

**Tool calls:**
1. Grep for stale recipe-vocabulary phrases ("tapestry or darn", "three recipes" and similar), to find prose tables the diff left unchanged. My context held only the diff.
2. Read `finalizerow.go` and `bouncerrows.go`, and grepped `RecipeDarn` use sites, to confirm sampler rows still resolve through the tapestry and darn lookups. The manifest did not show these files.
