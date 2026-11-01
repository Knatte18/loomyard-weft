```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens for round 1. For each change, I list what reads or asserts it.

**Per change: set found, covered or not**

1. **New gate name `clean`.**
   - The resolver `internal/shedrecipe/entries_gate.go`, its doc comment, the error text and the test rows are updated.
   - The darn, sampler and tapestry recipe YAML and `loomrecipe/darnrecipe_test.go` are updated.
   - `contracts/specs/shed-recipe-spec.md:73` still says "The vocabulary is `discussion`, `plan`, `description` and `verify`." It already omitted `parts` before this diff, but the diff widens the gap, and the spec lists no `clean`. This is a rule document describing the changed vocabulary, so it is not covered (F1 below).
   - The tapestry row and the sampler row take `WorktreeRoot` from `Env`, as `verify` does. I found nothing else that lists gate names.

2. **Fixer dirty path: `judge`, `dirtyPaths`, the `fixerEndDirty` meaning.**
   - `landingshed/doc.go` gets a line, and the `fixerEndDirty` and `check` doc comments are updated.
   - `haltReason` at `fixerrun.go:327-329` still words the dirty end as "ended without a usable turn (dirty)". That is still accurate, and the updated test asserts "(dirty)".
   - `verifygate.go` keeps `clean()` for its other callers, so existing callers are unchanged.
   - `contracts/stencils/landing/landing-template-fixer.md` has no dirty-tree rule. It has no wording that this diff contradicts, so this is covered.

3. **`verifytree.UncommittedFindings` and the `verifytree/doc.go` paragraph.**
   - Both are used by `cleangate.go` and `fixerrun.go`.
   - The doc paragraph is present, so this is covered.

4. **`internal/loomshed/doc.go` clean-gate paragraph.**
   - It is present. The sentence "A row lists it before its verify gate" is presented as a rule, but the sampler and darn Drift-Verify rows do not list it.
   - Drift-Verify (`darn-recipe.yaml:123`) is a gated agent row that edits the worktree and does not get `clean`.
   - That is a coverage-scope question for the handler's scope lens, not a stale doc. I note it only as the set not covered (see F2).

5. **PATTERN.md entries and `pattern/` background files.**
   - The diff touches neither.
   - The rubric says to raise nothing when the directive declares none. The constraints section lists no declared changes, so there is nothing to land.
   - I found no PATTERN entry the diff violates:
     - `PATTERN-gitrepo-client-boundary`: `NewCleanGate` calls `verifytree.DirtyPaths`, which uses `gitexec.Run`.
     - `PATTERN-test-speed`: the new test sits in a `_integration_test.go` file that calls `tiergate.Require(t, 1)` first. It also uses `t.Parallel()`.
     - `PATTERN-durable-vs-ephemeral-state`: the test writes `.scratch`, which is a literal, but it sits in a test and does not name `.lym` or `_lym`.

6. **Commit messages.**
   - The second commit says the clean gate is "listed first on Darn-Write and on every Code-Burler-Fix row". That matches the diff: darn, sampler and tapestry Code-Burler-Fix, plus Darn-Write.
   - No `Pattern-Departure:` trailer is present or needed.
   - The first commit ("Hold writer and fixer sessions…") says the session is re-prompted "so a stray file no longer reaches Finalize". That is slightly broad, since Webster and Drift-Verify sessions are not held. I count it as the same gap as F2.

**Findings**

- **F1 — LOW, class `consistency`**: `contracts/specs/shed-recipe-spec.md:73` lists the gate vocabulary without `clean` (and still without `parts`).
  - Evidence: that line reads "The vocabulary is `discussion`, `plan`, `description` and `verify`." The diff adds a name to the vocabulary without updating the spec.
  - Fix: add `clean` there, with one sentence on what it does and that it requires the absolute `Env.WorktreeRoot`.

- **F2 — MEDIUM, class `scope`**: the diff does not cover the producing rows that run a session and edit the worktree, and it states no reason.
  - Evidence: `contracts/recipes/darn-recipe.yaml:115-124` (Drift-Verify, gated on `verify` only). The board entry's Idea names "darn writer, Webster fork, fixers". The diff touches no Webster end gate (`internal/websterengine/verifygate.go:225` has its own `dirty` seam) and no Drift-Verify row.
  - I did not read the Webster fork's end-gate behaviour on a dirty tree. `websterengine/verifygate.go:395` shows it checks `len(dirty) > 0`, so it may already hold or halt there.
  - Fix: either wire the clean gate or the equivalent re-prompt into those rows, or record in the decision record and the `loomshed/doc.go` clean-gate paragraph why they are left out.

**Tool calls (3):**
1. A grep for `fixerEndDirty`, "(dirty)" and the gate-vocabulary phrasing across Go, md and yaml, because the manifest did not show other readers of the dirty-end text or vocabulary lists.
2. A grep listing the files that mention `gates:` or gate names, to find rule documents and stencils the manifest lacked.
3. Read `contracts/specs/shed-recipe-spec.md:70-80`, grep `darnengine` docs for gate wording, and grep the landing stencils for dirty-tree rules, because those were the candidate stale documents.
