```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals the fixer carried out as proposed (F1, F2, F3, F6, F7, F9, F10, F11, F12, F13).

Proposals challenged: F1, F2, F3, F6, F7, F9, F10, F11, F12, F13. F4 was fixed differently, so it is out of scope.

**MEDIUM — `contracts/stencils/loom/loom-rubric-code-review.md:90` (F7)**
- The reworded clause turns on "in a darn run", but the rubric's header says it is recipe-agnostic: "read it, in the tapestry, darn and sampler recipes alike. Each recipe sets the perch's answer key … never through this rubric." Lines 2 and 5.
- All three recipes load it through `rubric_stencil: loom-rubric-code-review`: `darn-recipe.yaml:182` and `:219`, `tapestry-recipe.yaml:395` and `:448`, `sampler-recipe.yaml:137` and `:175`.
- Nothing in the rubric tells the reviewer which recipe it is in. A reviewer with no recipe name cannot apply the rule. It may apply it everywhere, which is the original defect, or nowhere, which drops the check in darn.
- I did not check whether the review prompt names the recipe. If it does not, the scoping belongs in the darn recipe's answer key (`fasit`), not in the shared rubric.

**LOW — `internal/gateslot/runlog.go`, `NamedFailedTests` (F3)**
- It counts any `  <pkg> <Test>:` line under any run header and sets `Tier` to -1 for a non-tier run, such as `vet tier1`. The existing `FailedTests` test row "a vet run names none" shows such a line is possible.
- `judgeRedFirst` ignores `Tier`, so a vet or build-step line naming a changed test would count as red.
- `red-first` runs only tiers today, so this is latent. The `NamedFailedTests` doc comment does not say that non-tier-run lines are counted.

**LOW — `internal/gatecli/redfirst.go`, `run` (F13)**
- `red_tests` is built by appending each group's red tests in package first-seen order. The `redFirstVerdict.red` comment promises "the order of the changed tests", which holds only while `c.tests` is already grouped by package.

**LOW — `internal/gatecli/redfirst.go`, scoper branch (F2)**
- The scoper refusal is non-terminal in `redFirstFailure`, so on a `none` or `dotnet` hub every bug run burns all its attempts until the parent records `No test seam`. This is the proposal working as intended, but the cost goes unstated. Not a defect.

F1, F6, F9, F10, F11 and F12: no breakage found.

Tool calls (2):
- One `grep` of `loom-rubric-code-review.md`, to see whether the rubric names the recipe; my context held only the edited line.
- One `grep` of `contracts` for `rubric_stencil: loom-rubric-code-review`, to find which recipes share the rubric.
