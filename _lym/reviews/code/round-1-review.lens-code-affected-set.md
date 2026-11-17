```yaml
lens: code-affected-set
outcome: reported
```

**Task:** the code-affected-set lens. For each change the diff makes, I list the set I found that reads or asserts it, and report any member left wrong.

**Per change: set found, covered or not**

- **Plan-conflict path moved to `.lym/implementer/plan-conflict.md`** (`runplan.ConflictDir`).
  - Set: `contracts/recipes/tapestry-recipe.yaml:260`, `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml:364`, `contracts/specs/implementer-spec.md:44`, `contracts/specs/webster-spec.md:40`, `internal/websterengine/doc.go:828`, `internal/shedadapters/weaver.go` (`claimConflict`).
  - Result: covered. No `.lym/webster/plan-conflict.md` reference is left. `internal/websterengine/template_test.go:1028` only passes a path string as input to a prompt renderer.
- **Implementer spec: `weaver` table entry, Out line about committing during the run, stop-chain route.**
  - Set: `implementer-spec.md` and `webster-spec.md`.
  - Result: covered.
- **PATTERN files.**
  - `PATTERN-config-layers` gains `weaverengine`, `PATTERN-gate-slot` gains the two Weaver verify sites, `PATTERN-test-speed` gains the Weaver rows. These match the diff.
  - Result: `PATTERN-cwd-resolution` line 38 is not covered (finding 1).
  - Result: no `Pattern-Departure:` trailer exists on any of the 27 commits, so nothing is excused.
  - Result: the directive's constraints section declares no PATTERN changes, so there is nothing to check for landing.
- **Plan-commit pathspec** (`loomcli/wiring.go:150-165`) includes Weaver's directory. Covered by `TestPlanCommitPathspec`.
- **`websterengine.Dir` and `LoadState` readers outside webster.**
  - Set: `loomcli/wiring.go:155`, `loomcli/wiring.go:967`, `weaverengine/active.go:71,84,141,165`, `hubgeom` and `standalonegeom` webster geometry.
  - Result: the `loomcli/wiring.go:967` webster commit is Webster-only, which is correct.

**Findings**

1. **MEDIUM, class `consistency`.** `pattern/PATTERN-cwd-resolution.md:38` lists the engine packages that must not import `lymcwd`.
   - It names `websterengine` and `burlerengine` but not `weaverengine`, which the diff adds as an engine taking absolute paths.
   - The rule is marked `(test)`, so I'd expect an enforcement list that skips the new package.
   - Evidence: `pattern/PATTERN-cwd-resolution.md:38`; `git diff --stat` shows no edit to that file.
   - I did not read the enforcing test, so confirm whether the list is a closed set the test checks.
2. **LOW, class `design`.** The Plan-Review text in `contracts/recipes/tapestry-recipe.yaml:260-262` now says "the implementer's begin", but the next lines still say fixes are made by "editing only cards Webster's auto-rebaseline accepts".
   - Under Weaver, D7 and `adoptPlan` (`internal/weaverengine/run.go:365-384`) apply a different rule: cards whose batch is done are frozen.
   - The decision record's acceptance criterion asks that the Plan-Review text name the shared path. It does not ask for the card rule to be generalised, so this is a wording gap, not a violation.
   - Evidence: `contracts/recipes/tapestry-recipe.yaml:260-263`.

**Tool calls (3 of 6)**
1. `grep` for `plan-conflict` references and Weaver mentions in docs and patterns, plus `websterengine.Dir/LoadState` callers. The manifest held only the diff head, not these callers.
2. `sed` of `loomcli/wiring.go` (150-165, 955-975) and `websterengine/doc.go` (820-832). I needed to check that the pathspec and the Webster-only commit were correct.
3. `grep` of `pattern/*.md` and the specs for webster-specific enumerations, to find PATTERN files that should now name Weaver.
