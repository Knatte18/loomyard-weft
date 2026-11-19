```yaml
lens: code-correctness
outcome: reported
```

Scope: the code-correctness lens on the diff's changed functions and branches.

**Functions and branches, the case run for each, and the verdict**

- **`prReviewSendBackEdge`, error from `loomrecipe.EngineRow`.** Given an unknown recipe name, it returns `false, "", err` before any I/O, and the verb wraps the error through `prFindingsReadFailure`. Correct.
- **`prReviewSendBackEdge`, `hasRow == false`.** The pawl recipe has no PRReview row; only the darn, sampler and tapestry recipes carry one (`contracts/recipes/*.yaml`). It returns `false, "this run's recipe has no PR-Review row", nil` without opening the fabric, so the pawl-only `primeOfHub` and `beamParentWorktree` branch of `wireLanding` is never reached. Correct.
- **`prReviewSendBackEdge`, `wireLanding` fails.** Two failures were run: `fabricengine.Open` fails, and `ReadOrigin` or `resolveLandingParent` fails in `landingBranches`. The error comes back unwrapped, and `prFindingsVerb` reports it as "read whether the send-back edge can fire: …" with a way forward. That happens before `commit` and before any write the verb makes after the edge, so no state is left inconsistent. Correct.
- **`prReviewSendBackEdge`, `wireLanding` succeeds on a tapestry run.** The Finalize and ParentAlign constructors check `Deps.OpenFabric`, and `landingDeps` now sets it (`landingdeps.go:115`), so the nil refusal behind the reported error is gone. `newShed` then builds, and `sendBackEdge` reads `OnReroute`, the entry gate and `MaxBounces` as before. Correct.
- **`prReviewSendBackEdge`, `newShed` fails after wiring.** The error is returned and `c.env.Landing` stays filled. The CLI process is per invocation and the edge runs once per verb call, so the leftover state is never read again. Correct.
- **`prReviewSendBackEdge` run twice, or after `wire`.** `wire` never fills `c.env.Landing`; only `wireLanding` does (`sharedbootstrap.go:404-464`, `arm.go:517`, `buildLoomShed`). A second call overwrites the fields with equal values. It does open a second sampler record watch on a sampler recipe (next item). Correct otherwise.
- **`prReviewSendBackEdge` on a sampler run.** `sampler-recipe.yaml` holds both PRReview and SamplerSession, so the edge now enters the `recipeHasEngine(c.recipe, samplerSessionEngine)` branch. That branch runs `openRecordWatch`, which calls `os.MkdirAll` on the record's directory and starts a file watcher whose closer is dropped (`samplerwiring.go:91-98`). The old edge did neither. The SamplerSession constructor may need these seams for the shed to build, so filling them is warranted. The watcher lives only until the short-lived verb process exits. LOW at most (finding 1).
- **`prFindingsCmd` with `edge: c.prReviewSendBackEdge`.** The method value binds `c` when the command runs. `c` is a pointer and the pre-run has already armed it, because `verbUsesLightweightWiring` omits `pr-findings`, so the full `wire` runs and sets `reed`, `runner`, `registry` and `shedPaths`. Correct.
- **Test, pre-fix behaviour.** Without the `wireLanding` call, `c.env.Landing.OpenFabric` is nil and `loomrecipe.New` fails at the tapestry Finalize row with the issue's error, so the test fails. With the call it passes. Correct.

**Findings**

1. **LOW · design · `internal/loomcli/prfindings.go:298`.** On a sampler-recipe run, `prReviewSendBackEdge` calls `wireLanding`, which also opens the sampler record watch: a `MkdirAll` and a file watcher that is never closed. Building the edge's shed does not need the watch, and the old edge never opened it.
   - Evidence: `internal/loomcli/sharedbootstrap.go:443-448` (the `recipeHasEngine(c.recipe, samplerSessionEngine)` branch calling `openRecordWatch`) and `internal/loomcli/samplerwiring.go:91-98`. `contracts/recipes/sampler-recipe.yaml` holds both PRReview and SamplerSession.
   - Impact is limited to the lifetime of one verb process and a directory a live sampler run already holds.

No other correctness defect found.

**Tool calls**
- grep of `internal/loomrecipe/loomrecipe.go` and `names.go` for the recipe rows. The context did not show which recipes carry PRReview or SamplerSession.
- A failed shell search, an empty `dirname` that made grep walk `/`, run to find the embedded recipe directory.
- grep of `internal/loomrecipe` and `internal/shedrecipe` yaml for PRReview and SamplerSession. It found only test data and the import path of the recipes package.
- grep of `contracts/recipes/*` for PRReview and SamplerSession, to learn which recipes the edge can reach and whether `wireLanding`'s sampler or pawl branch runs.
