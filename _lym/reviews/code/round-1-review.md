---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/specs/shed-recipe-spec.md:73
    summary: The shed recipe spec lists the gate vocabulary without the new clean name, so the contract document no longer matches the resolver.
    fix: In contracts/specs/shed-recipe-spec.md line 73, change the vocabulary sentence to include `clean` (and the already-missing `parts`), and add one sentence that `clean` fails while git reports an uncommitted or untracked path in the worktree and requires the absolute `Env.WorktreeRoot`.
  - id: F2
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/shedrecipe/entries_gate_test.go:348
    summary: The clean gate's refusal of a relative or empty WorktreeRoot has no test row, though the verify and discussion gates have one.
    fix: In internal/shedrecipe/entries_gate_test.go, add a row for the `clean` gate to the table-driven test that refuses an unset or relative root for the verify gate (around line 348), asserting the error names `WorktreeRoot`.
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/loomrecipe/darnrecipe_test.go:288
    summary: No recipe test pins the clean gate on the tapestry Code-Burler-Fix row, so deleting that entry would break nothing.
    fix: Add a case to a tapestry recipe test in internal/loomrecipe (one that already parses `recipes.TapestryRecipe`) asserting the Code-Burler-Fix row lists `clean` and then `verify` in its gates, or extend the existing gate-list assertion to the tapestry recipe.
---

### [LOW] Gate vocabulary spec omits `clean`

**Location:** contracts/specs/shed-recipe-spec.md:73
**Issue:** The spec says "The vocabulary is `discussion`, `plan`, `description` and `verify`." The diff adds `clean` to the closed vocabulary in `internal/shedrecipe/entries_gate.go` and to three shipped recipes, but the document that defines the recipe contract was not updated. It already omitted `parts` before this diff, so the gap widens rather than opens.

### [LOW] Clean gate's absolute-root refusal is untested

**Location:** internal/shedrecipe/entries_gate_test.go:348
**Issue:** `resolvePlainGateClosure` runs `requireAbsRoot` for the `clean` case, as it does for `verify` and `discussion`, but only those two have a refusal row. A removal of the check would pass the suite.

### [LOW] Tapestry Code-Burler-Fix clean entry is unpinned

**Location:** internal/loomrecipe/darnrecipe_test.go:288
**Issue:** `TestDarnVerifyBudget` pins `clean` on the darn and sampler rows. The tapestry Code-Burler-Fix row at `contracts/recipes/tapestry-recipe.yaml:556` has no recipe-level assertion.

## Rejected

- Fork code-correctness F1 (escalate report on a dirty tree is re-prompted first): the dirty check already ran before the report check, and a dirty tree ended the row before this diff; the diff makes it no worse and the fixer can commit or delete before escalating.
- Fork code-correctness F2 (unbounded path list in `UncommittedFindings`): `git status --porcelain` collapses an untracked directory to one entry, and the findings go to a findings file, so a runaway list is not a realistic path.
- Fork code-correctness F3 (`pass_on_cap` accepted on `clean`): `pass_on_cap` is a generic per-entry setting every gate takes, no shipped recipe sets it on `clean`, and the loader treats it the same for every gate; refusing it for one name is a design change the board entry does not ask for.
- Fork code-affected-set F2 (Webster, Drift-Verify and other producing rows left out): each already holds its session on a dirty tree. `internal/websterengine/verifygate.go:391-396` fails the end gate on dirty paths, `internal/driftengine/verify.go:33` and `:422` put a `dirty` entry first in Drift-Verify's table, and the sampler session and PR-Review check dirty paths in `internal/loomshed/samplersession.go:263` and `internal/landingshed/prreview.go:459`. The darn writer was the gap because the darn-form verify gate returns passed before `verifytree.Verify` when the tree-scoped command is empty (`internal/loomshed/verifygate.go:117`), skipping its dirty check.
- Fork code-tests F-tests-3 (gate order not asserted): the order is recipe data; a reordering still re-prompts a dirty tree through the verify gate's own dirty finding, so no behavior breaks.
- Fork code-tests F-tests-4 (git-status failure path untested): both paths return the error from `verifytree.DirtyPaths` unchanged; the git boundary is the trusted dependency, and the same path is covered where `verifytree` calls it.
- Fork code-tests items 12 and 13 (`dirtyPaths` error and nil seam): behavior carried over unchanged from `clean`, which the diff only refactors.
