```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set-based findings' searches (F6, F7, F8, F1–F3) and check the fix covers every member.

No MEDIUM or BLOCKING findings. One LOW:

- **LOW** — `internal/runprofile/template.yaml:3`, a sibling of F6's fix. It describes `loom.raddle_auto_approve` as telling a raddle run to file without waiting for the operator's approval. The fixed `template.yaml` comment and `config.go:612` now add "never read in a run whose recipe rows hold a split perch", but this third description of the key does not. That is stale only if the profile's raddle block is meant to say the setting is inert for split-perch runs. The same applies to `internal/loomengine/raddlewiring.go:39`, which says "When it is true, a raddle run files without waiting for the operator's approval".

Searches that found no further members:

- **F6:** the only statements of "read once, at Raddle-Write's first spawn" are `config.go:612` and the `template.yaml` comment, and both now carry the perch clause. The other hits (`raddlewrite_test.go:130`, `raddlestate.go:24`) describe mode recording, not the key.
- **F7:** other hits naming the operator as approver are `contracts/recipes/raddle-recipe.yaml:1,3`, `contracts/specs/raddle-proposal-spec.md:25`, `internal/shedrun/seed.go:82`, `internal/loomrecipe/doc.go:50`, `internal/loomcli/recipekeyed_test.go:50` and `internal/loomshed/raddlefile.go:205`. These describe the raddle recipe or interactive mode, which is never in perch mode, and the review rejected the recipe-header variant. No other stencil heading is in the set.
- **F8:** `recipekeys.go:159` is the only place that compares an `artifact_paths` entry to `RaddleProposalRel()`, and it is now cleaned.
- **F1–F3:** `perchFilingRefusal` has one caller (`decision.go:745`). `raddleRunFiling` has one non-test caller (`decision.go:1120`). `raddleBoardContext` has one non-test caller (`raddlecontext.go:127`, via `raddleCheckFor`). The added tests cover each of them.

Tool calls (all read-only greps; my context did not hold the repo-wide result sets):
1. Grep for "first spawn" across `.go`, `.yaml` and `.md`, to find every statement of the key's read timing (F6).
2. Grep for "operator approves" and similar wording in raddle text, to find sibling approver phrasing (F7).
3. Grep for `RaddleProposalRel()` and `FromSlash(` on proposal and artifact paths, to find other path comparisons (F8).
4. Grep for callers of `perchFilingRefusal`, `raddleRunFiling` and `raddleBoardContext(` outside tests, to check the coverage sets (F1–F3).
