```yaml
lens: repo-invariants
outcome: reported
```

Lens: repo-invariants. For each card I checked three things: reflective seam tests, doc text the card falsifies, and whether the card's own gate can pass from the state earlier cards leave.

**Per card:**
- **Card 1:** Seams held (`TestNewShed_FieldCompleteAssembly` listed). Doc broken: the second retired-row mechanism leaves loomcli's doc saying retired rows are read only through `loomrecipe.RetiredRows` (F1). Order held.
- **Card 2:** Seams held (`TestLandingAndApprovalAccessors` listed). Docs held (pawlengine `doc.go` extended). Order held.
- **Card 3:** Seams held (both `pullrequest.API` fakes listed; envkit and loomcli implement none). Docs held. Order held.
- **Card 4:** Seams held. Docs held (landingshed `doc.go` listed). Order held.
- **Card 5:**
  - Seams held: `shedrecipe#Env`, `pawlengine#Deps`, `intentionallyNil`, `FullEnv` and the seam tests are listed.
  - Doc broken: the hold boundary moves to Finalize, but `internal/dobbyengine/doc.go:119` and the `runTakesNoSlot` comment at `internal/battencli/dobbyreads.go:214` still say "from Parent-Align on" (F2).
  - Doc broken: `RetiredRowNames` sits beside the existing `loomrecipe.RetiredRows` with a different source and meaning, and no doc says so (F1).
  - Order held: `beamwiring.go` stays uncalled; unexported dead code compiles, and no repo test or gate flags unused code (grep found none).
- **Card 6:** Seams held (`PullRequestDeps.Reopen` is dropped from `intentionallyNil` and envkit; `loomcli/landingdeps.go:385` sits inside `pullRequestDepsGuardedBy`, which is listed). Docs held (beamcli `cli.go` reopen help goes with `reopenCmd`; hubgeom `PawlPaths` is listed). Order held.
- **Card 7:**
  - Seams held: `dobbyengine.Deps` has no reflective seam test.
  - The dobbyengine import allowlist holds: no new import is needed.
  - Docs held for its own change. Order held.
- **Card 8:** Seams held (`shedbuild.ShedPaths` fields are function seams, assembled by card 1's test). Docs held. Order held.
- **Card 9:** Seams held (`sheddriver.Deps` is not seam-reflected). Docs held (extends loomshed and shedrun `doc.go`). Order held: an absent wake stamp before card 10 reads as none.
- **Card 10:** Seams held (`pawlengine#Deps` fields filled in `FullEnv`, `pawlDeps` and `validPawlDeps`; the wire seam tests are listed under Uses). Docs held. Order held.
- **Card 11:** Seams held (`PullRequestDeps` new fields are listed in loomcli and battencli `intentionallyNil` and envkit `LandingDeps`). The `PATTERN.md` edits are carried. Order held.
- **Card 12:** Seams held. Docs held (loomcli and fabricengine `doc.go` listed). Order held.
- **Card 13:** Seams held. The `PATTERN-github-auth` edit is carried. Docs held. Order held.
- **Cards 14, 15, 17, 18:** Seams held (`landDeps` has no reflective test). Docs held. Order held.
- **Card 16:** Seams held. The dobbyengine `TestToldGeometryInvariant_AllowlistOnly` holds: the path is joined in hubgeom and no new import is needed. Order held.

**Findings:**

- **F1.** MEDIUM, consistency.
  - Location: card 1 (`shedbuild#Recipe`) and card 5 (`loomrecipe#RetiredRowNames`).
  - Summary: The repository already has a retired-row mechanism. `loomrecipe.RetiredRows` is a per-recipe Go table, and `loomCLI.renameRetiredRows` applies it at arm time. Cards 1 and 5 add a second mechanism (the recipe YAML `retired:` map plus `RetiredRowNames`) without naming the first. That makes two package docs false or incomplete:
    - `internal/loomcli/doc.go:11` says retired row names are "read through `loomrecipe.RetiredRows`".
    - `internal/loomrecipe/doc.go:39` describes `RetiredRows` as each recipe's retired-row data.
  - Cards 5 and 6 list both `doc.go` files, but neither Intent addresses this. `RetiredRowNames` reads a different map than its near-namesake.
  - Evidence: `internal/loomcli/arm.go:288`, `internal/loomcli/darnwiring.go:176-178`, `internal/loomrecipe/recipekeys.go:20-22`, `internal/loomcli/doc.go:11`, `internal/loomrecipe/doc.go:39`.
- **F2.** LOW, consistency.
  - Location: card 5 (hold boundary moves to Finalize).
  - Summary: Two texts still say a pawl run takes a `max_runs` slot "from Parent-Align on", and no card's Intent rewords them:
    - `internal/dobbyengine/doc.go:119`. Cards 7 and 16 edit this file, but for other reasons.
    - The `runTakesNoSlot` doc comment at `internal/battencli/dobbyreads.go:214`. No card lists this file.
  - Evidence: `internal/dobbyengine/doc.go:119`, `internal/battencli/dobbyreads.go:214`.

**Tool calls:**
1. Read the dobbyengine and landingshed seam-enforcement test headers and the `//lym:seam` test files, to learn which structs are reflected; my context lacked the seam-test inventory.
2. Listed the structs named by `//lym:seam` markers and grepped docs for old-flow text, to map reflective coverage and stale docs; my context lacked both.
3. Read the stale lines in the dobbyengine, loomrecipe and boardengine docs, the function that owns `loomcli/landingdeps.go:385`, and the dobbyengine import-allowlist test, to verify doc text and card 6's coverage.
4. Grepped `RetiredRows` and `NoSlot` sources, after the loomrecipe doc revealed an existing retired-row mechanism that my context lacked.
5. Grepped non-test Go for hold-boundary and reopen text outside the files the cards list, to find doc text no card lists.
