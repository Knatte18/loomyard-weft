```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens over the diff's changes.

**What each change reaches, and whether the diff covers it**

1. **`pr-findings`' send-back edge now calls `wireLanding` before `newShed`.** Before this diff, only `run` and `step` passed through `wireLanding`. This is new behaviour for `pr-findings`: it opens the fabric, reads the landing branches and, on a sampler or pawl recipe, runs those branches.
   - `internal/loomcli/doc.go:42`: updated. Covered.
   - `internal/loomcli/doc.go:605`: new line, accurate. Covered.
   - `internal/loomcli/arm.go:467-469`: these lines say each of `run` and `step` calls `wireLanding` once. They are still true, and `pr-findings` builds no `BuildShed`. Covered.
   - `internal/loomcli/wiring.go:1232, 1317`: these say `wireLanding` fills `Env.Landing` and the reed seams. Still true. Covered.
   - `internal/loomcli/sharedbootstrap.go:436-439`: these lines say "The building paths pass through here; status and pause never do. So the watch opens only for a run that drives the row." `contracts/recipes/sampler-recipe.yaml` has both a SamplerSession row and a PRReview row. So `pr-findings` on a sampler run now passes the PR-Review row check and calls `wireLanding`, and that opens the sampler record watch without driving the row. The comment is now false. **Not covered.**
   - `_lym/landing/summary.md`: it says the edge skips `buildLoomShed` because that path opens a pawl run's land-request watch, which `pr-findings` does not need. On a sampler run, though, the edge now opens a different watch through `wireLanding`, and the summary does not say so. This is not a false claim, so it is no finding.
   - The unit tests of `prFindingsVerb` inject a fake `edge`, and the field's type is unchanged. Covered.
2. **The edge closure becomes the method `prReviewSendBackEdge`.** The only call site is `prFindingsCmd`, which this diff updates. Covered.
3. **New test file.** It follows the rule that each file names one tier, and it relies on `testmain_test.go` for the hermetic git setup. Covered.
4. **Commit messages, PATTERN and stencils.** No commit carries a `Pattern-Departure:` or `Loosened-Check:` trailer, the directive declares no PATTERN changes, and no stencil changed. Nothing applies.

**Findings**

- **LOW, consistency.** `internal/loomcli/sharedbootstrap.go:436-439`: the comment says only building paths pass through `wireLanding`, so the sampler record watch opens only for a run that drives the row.
  - Since this diff, `pr-findings` also passes through it. On a sampler run, which has both rows per `contracts/recipes/sampler-recipe.yaml`, that sets the Sampler seams and opens the watch through `openRecordWatch` (`samplerwiring.go:91`).
  - Evidence: `internal/loomcli/prfindings.go:298` and `internal/loomcli/sharedbootstrap.go:438-445`.
  - Fix: reword the comment so it names `pr-findings`' edge as a pass-through that does not drive the row. Or stop the edge from opening the watch.

**Tool calls**
1. Grepped the repo for `wireLanding`, "building paths" and "send-back edge" in Go and Markdown, and read `doc.go` lines 30–50 and 598–615. My context did not hold the other descriptions of `wireLanding`'s callers.
2. Grepped `loomrecipe` and `shedrecipe` for SamplerSession. My context did not show which recipe holds that row.
3. Grepped the recipe YAML files for the SamplerSession and PRReview rows, to learn whether one recipe holds both.
