---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/07-darn-gate-impact-scope.md:59
    summary: darnTreeScope returns darn.yaml load errors raw, so a malformed tree key reaches the writer gate, Merge-In, Finalize and Publish with no way forward, against PATTERN-refusal-way-forward
    fix: >-
      In Card 7's `darnTreeScope` paragraph, replace "every error is returned" with:
      "a `darnengine.LoadConfig` error is returned through `darnengine.WithEditorWayForward`, and every other error, the merge-base and diff reads included, is returned as is".
      In the same card's `TestDarnTreeScope_ClassifiesAtEachCall` step "with the hub `darn.yaml` holding `./tools` under `tree_scope`, the seam returns an error naming `./tools`", append "and `lyx config darn`'s editor".
      In Card 8, add to the `darnVerifySource` paragraph that the command's tree-scope error carries that editor way forward because it comes from `darnTreeScope`, so Merge-In's and Finalize's "read verify command" error and Publish's seam error name it.
      Card 7 already lists `plan:internal/darnengine#WithEditorWayForward` under `**Uses:**`, so no target changes.
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/03-editor-save-load-check.md:32
    summary: Card 3 picks the clear-first editor way forward for every no-tree load error, but D1 gives it only for a last tree removed, so a save that turns tree scope on with no tree gets a step that cannot succeed
    fix: >-
      In Card 3, replace the sentence beginning "When the loader's error wraps `darnengine.ErrTreeScopeWithoutTrees`" with:
      "When the loader's error wraps `darnengine.ErrTreeScopeWithoutTrees` and the hub file before the save held a tree that the staged file holds none of, the input being a save that empties `tree_scope` while `tree_scope_verify` stays non-empty, the way forward is to run `lyx config darn --set tree_scope_verify=` first, then remove the trees in `lyx config darn` again.
      Every other load refusal, a no-tree save over a file that held no tree included, takes the copy-out way forward below."
      Add one row to the darn rows of `TestDispatchHubWideBoard`, over a board `darn.yaml` that is the template.
      In that row, a save setting a non-empty `tree_scope_verify` holding `{targets}` while `tree_scope` stays empty is refused with exit 1.
      The board file is unchanged, the staging copy holds the saved bytes, and the error names the loader's no-tree text, the staging path and the copy-out way forward, and not `--set tree_scope_verify=`.
  - id: F3
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/plan/00-overview.md:38
    summary: the plan verify runs neither ./contracts/specs, the package of Card 7's shed-recipe-spec.md target, nor ./contracts/stencils, whose stencil tests scan Card 9's darn-template-write.md
    fix: >-
      In `00-overview.md`'s `## verify:` block, append ` ./contracts/specs ./contracts/stencils` to the end of the `lyx gate test --tags integration ...` line, after `./cmd/lyx`.
      `lyx loom validate-plan --overlay` with this change and F7's together reports no finding.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/02-darn-tree-scope-config.md:43
    summary: WithEditorWayForward's fixed clause ends with resume the run, but RecipeOverride refuses at lyx loom start's seed stage before any seed exists, where resume refuses in turn
    fix: >-
      In Card 2's `WithEditorWayForward` sentence, change the appended clause to
      "fix the named tree_scope key in `lyx config darn`'s editor from the prime, then re-run the lyx loom command that refused".
      In Card 2's `TestVerifyCommand` row and Card 7's `TestWire_DarnRecipe` subtest and `TestRecipeOverride` row, assert that clause.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/02-darn-tree-scope-config.md:40
    summary: the no-tree refusal and the tree_scope template comment name --set tree_scope.<tree>=<target> as the way to add a tree, which cannot write a tree path holding a dot, since --set splits its key path on every dot
    fix: >-
      In Card 2, add to both the `tree_scope` template comment sentence and the no-tree refusal bullet:
      "a tree path holding `.`, such as `.github`, is written in `lyx config darn`'s editor".
      Extend the no-tree row of `TestLoadConfig` to assert that clause.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/02-darn-tree-scope-config.md:50
    summary: TreeScopedCommand re-trims a trailing slash and treats an empty tree_scope as off, two states LoadConfig already rules out for its only caller, and two TestTreeScopedCommand rows pin them
    fix: >-
      In Card 2's `TreeScopedCommand` paragraphs, drop "a tree key's one trailing `/` is trimmed before the match", and change "when `TreeScopeVerify` or `TreeScope` is empty" to "when `TreeScopeVerify` is empty".
      Say that the function is told a config `LoadConfig` returned.
      From `TestTreeScopedCommand`'s table, drop the rows "a key `tools/` claims `tools/a.go`" and the "with no tree" half of "tree scope off".
      The `tools/` criterion stays pinned by `TestLoadConfig`'s "a key written `tools/` loads as `tools`" row together with the "a path equal to a tree path" and `tools` rows.
  - id: F7
    severity: LOW
    class: scope
    origin: lens:affected-set
    location: _lyx/plan/07-darn-gate-impact-scope.md:14
    summary: contracts/recipes/darn-recipe.yaml says the Darn-Burler-Fix verify gate runs darn.yaml's verify command after each round, which Card 7 makes false, and no card targets it
    fix: >-
      Add the bullet `contracts/recipes/darn-recipe.yaml` to Card 7's `**Edit:**` list after `contracts/specs/shed-recipe-spec.md`.
      Add one sentence to Card 7's Intent: the Darn-Burler-Fix row's gate comment in `darn-recipe.yaml` is restated to say the gate runs the tree-scoped command, or the command `impactset` derives from the merge base, and runs darn.yaml's `verify` where `impactset` falls back.
      The overlay check reports no finding for this change.
  - id: F8
    severity: LOW
    class: design
    origin: handler
    location: _lyx/plan/05-derived-command-gate-test.md:19
    summary: writing each outside guard package as its own lyx gate test --tags integration step makes every one of them rebuild lyx, about 2s each warm, which works against D3's cost goal
    fix: >-
      In Card 5, write the guard steps as one step,
      `lyx gate test --tags integration <outside guard packages> -- -run '^(<names>)$'`,
      with the packages in sorted directory order and the names sorted and deduplicated, as webster's card-gate guard step already does (`internal/websterengine/guardstep.go`).
      Restate the overview's second Shared Decision to say so.
      Update `TestCommandFor`'s rows to expect the single step.
  - id: F9
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/plan/07-darn-gate-impact-scope.md:68
    summary: the Merge-In StatusSkipped step re-pins verifytree's same-tree, same-command skip, which TestVerify_Scenario and TestVerify_PerCommandRecord already cover
    fix: >-
      Drop the `TestVerifyGate_RoundScenario` darn-form step "right after that pass, with no failure record present, `verifytree.Verify` at site `Merge-In` ... answers `StatusSkipped`" from Card 7.
      The preceding step already pins that the gate records its pass under the scoped command string Merge-In resolves to.
---

The plan carries D1 to D6 and every Constraint of the decision record, and adds no scope the record does not license.
The blocking defect is in how the new per-call config reader reports errors.
Every other finding is a wording, coverage or economy fix.

### [BLOCKING] Per-call tree-scope reader drops the editor way forward

**Location:** `_lyx/plan/07-darn-gate-impact-scope.md:59`, carried into `_lyx/plan/08-darn-landing-tree-scope.md:27`

**Issue:** Card 2 builds `WithEditorWayForward` so that a malformed or duplicate tree key names `lyx config darn`'s editor.
Cards 2 and 7 apply it only at run start: in `VerifyCommand`, in `wireDarn` and in `RecipeOverride`.
`darnTreeScope` reloads `darn.yaml` at every writer-gate arrival, Merge-In, Finalize and Publish, and returns the load error raw ("every error is returned").
Card 8 puts it ahead of `VerifyCommand` in `darnVerifySource.command`, so the wrapping inside `VerifyCommand` never runs for this error at the landing sites.
The source rereads the hub file at every call, "so an edit between steps is seen" (`internal/loomcli/darnwiring.go:46-47`).
A Bolt edit made mid-run that the editor never checked therefore stops a `loom` verb with a refusal that names no way forward.
`PATTERN-refusal-way-forward` binds the `internal/loom*` packages, so this violates a listed constraint.

### [MEDIUM] Editor no-tree way forward is chosen by sentinel, not by D1's condition

**Location:** `_lyx/plan/03-editor-save-load-check.md:32`

**Issue:** D1 gives "run `lyx config darn --set tree_scope_verify=` first" only "for a last tree removed while `tree_scope_verify` is non-empty", and the copy-out way forward otherwise.
Card 3 keys the choice on the sentinel `ErrTreeScopeWithoutTrees`, which a second input also reaches.
That input is an editor save that sets a non-empty `tree_scope_verify` over a file whose `tree_scope` was empty and stays empty.
For that operator, clearing an already-empty `tree_scope_verify` is a no-op and there are no trees to remove.
They are also never told to copy the kept staging copy out before the next open overwrites it.
None of Card 3's rows covers that save.

### [MEDIUM] Plan verify misses two targeted contract packages

**Location:** `_lyx/plan/00-overview.md:38`

**Issue:** Card 7 targets `contracts/specs/shed-recipe-spec.md`, which lies in the Go package `contracts/specs` with its own tests (`contracts/specs/specs_test.go`).
Card 9 edits `contracts/stencils/darn/darn-template-write.md`.
The tests that scan that stencil for markers and banned text live in `contracts/stencils` (`markers_test.go`, `govalues_test.go`, `registry_test.go`).
The plan's `## verify:` runs neither package, and the plan-writer stencil's list of verify tiers is `none`, so the coverage rule applies.

### [LOW] Run-start way forward says resume where no run exists yet

**Location:** `_lyx/plan/02-darn-tree-scope-config.md:43`

**Issue:** `RecipeOverride` runs in `lyx loom start`'s seed stage, before `shedrun.WriteSeed` (`internal/loomcli/sharedbootstrap.go:110-116`).
On a first start, "then resume the run" sends the operator to `lyx loom resume`, which refuses because no seed exists.

### [LOW] `--set` way forward cannot write a dotted tree path

**Location:** `_lyx/plan/02-darn-tree-scope-config.md:40`, and `:32` for the template comment

**Issue:** `--set` splits its key path on every `.`, so `--set tree_scope..github=` writes a nested map instead of the tree `.github`.
Card 2's malformed-key rules accept such a key, so only the editor can write it.
Both texts name only `--set`.

### [LOW] Classifier guards states the loader already rules out

**Location:** `_lyx/plan/02-darn-tree-scope-config.md:50`, `:60`

**Issue:** `LoadConfig` already trims each key's trailing `/`, and it refuses a non-empty `tree_scope_verify` with no tree.
The only caller, `darnTreeScope`, passes `TreeScopedCommand` a loaded config.
The second trim, the empty-`TreeScope` off check and the two table rows pinning them therefore cover states that cannot occur, against `code-quality`'s rule against handling impossible states.

### [LOW] Darn recipe's gate comment goes stale

**Location:** `contracts/recipes/darn-recipe.yaml:155-157`, against `_lyx/plan/07-darn-gate-impact-scope.md:14`

**Issue:** Card 7 restates `loomshed/doc.go`, `shed-recipe-spec.md` and the `darnwiring.go` header for the new gate form.
It does not restate the recipe comment that describes the same gate.

### [LOW] One lyx build per guard package in the derived command

**Location:** `_lyx/plan/05-derived-command-gate-test.md:19`

**Issue:** `lyx gate test` with non-empty `--tags` runs `prebuildLyx` on every call (`internal/gatecli/run.go`, `internal/gatecli/prebuild.go:45-63`).
The repository holds `//lyx:guard` tests in 11 package directories, so a derived command can carry up to one extra `lyx` link per guard package.
A warm build of `./cmd/lyx` measured 1.9s here, and longer is likely on OSL-1046 under Cortex XDR.
Webster's card gate already runs every guard package in one `lyx gate test` call with a joined `-run`, which the Shared Decision's reason for avoiding `--guards` does not rule out.

### [LOW] Merge-In skip step duplicates verifytree's own tests

**Location:** `_lyx/plan/07-darn-gate-impact-scope.md:68`

**Issue:** The step calls `verifytree.Verify` directly and asserts its skip on the same tree and command.
That rule is pinned in `internal/verifytree/verify_integration_test.go` by `TestVerify_Scenario` ("a skip needs a record naming HEAD's tree and the same command") and `TestVerify_PerCommandRecord`.
The darn-specific part, that the gate records under the scoped string, is the preceding step.

## Rejected

- `TestDarnTreeScope_ClassifiesAtEachCall`'s `./tools` step as a repeat of `TestLoadConfig`'s malformed-key row: once F1's fix applies, the step pins the seam's own wrapping, which no other test covers.
- `ModuleGuardTests` as a second module-wide guard enumeration beside webster's walk-based `scanGuardTests`: D5 names `go list ./...` as the enumeration whose failure the verb reports, so the plan follows the decision.
- `configcli` gaining a darn-specific branch through `darnengine.ErrTreeScopeWithoutTrees`: D1 prescribes darn-specific editor text, and `darnengine` imports nothing from `configcli`, so no import cycle arises.
- Card 4's unreadable-file and unreadable-directory steps repeating Card 6's scenario rows: the writer stencil requires each card to carry coverage for the behaviour it introduces, and Card 6's rows are needed to reach `guardScanRefusal`'s branches.

## Focus verdicts

- Fidelity: every Decision D1 to D6 and every Constraint has a carrying card.
  Card 3 carries the editor save's sentinel, kept staging copy and own way forward, with the D1 condition defect in F2.
  Card 8 carries D4's per-call resolution at Merge-In and Finalize.
  No card adds unlicensed scope.
- Card order: no card depends on a later one.
  Card 7 creates `darnTreeScope` and `TestDarnTreeScope_ClassifiesAtEachCall`, and Card 8 uses or edits them.
  Card 5's command change comes before Card 7's tests that run it.
- `TestDarnTreeScope_ClassifiesAtEachCall` is planned once: Card 7 creates it and Card 8 extends it.
  The test-economy findings are F6 and F9.
- Reachability: every prescribed refusal has a named input and a test that reaches it.
  Card 7's empty-command and nil-merge-base branches are fail-loud wiring guards.
  The unreachable classifier branches are F6.
- Attack surface: the `--guards` Intent (Card 6), the Publish skip (Card 8), the impact-scoped form (Card 7) and D2's tree-scope bound in Card 2's `doc.go` each state what they let through and what bounds it.
- Re-sign arrows: `NewVerifyGate` carries one.
  `Publish.Call`, `darnVerifySource`, `runTest` and `landingDeps` keep their signatures.
  `Deps`, `verifySource` and `testRequest` are structs that only gain a field, and correctly take no arrow.
- CLI shape: `--guards` on the operator-audience `lyx gate test` is not ruled out by the audience bullet of `pattern/PATTERN-cli-cobra.md`.
