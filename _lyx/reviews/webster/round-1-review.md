---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/loomcli/darnwiring.go:263
    summary: wireDarn tells the verify gate darnVerifySource's per-call command, so the gate's told command and BaseCommand become the scoped command for a tree-scoped diff instead of the hub's whole verify
    fix: In wireDarn set c.env.VerifyCommand to a closure over darnengine.VerifyCommand(dirs) with dirs := fabricengine.ConfigDirs(location), as Card 7 says, and keep darnVerifySource(location).command for landingDeps only. Add the Card 7 assertion to TestWire_DarnRecipe in internal/loomcli/wiring_darn_test.go, a darn seed over a hub darn.yaml with tree scope on, where c.env.VerifyCommand() returns the hub's verify, not the scoped command.
    origin: lens:code-tests
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/darnengine/config.go:321
    summary: TreeScopedCommand credits a path only to the first sorted tree it lies in, so with nested trees the more specific tree's target is dropped and its tests do not run
    fix: In TreeScopedCommand mark every tree the path lies in as touched, not only the first match from slices.IndexFunc, and return ("", false) only when a path lies in no tree. Add a TestTreeScopedCommand row with nested trees, for example docs/empty target and docs/api with target ./api, a path under docs/api expecting both targets in tree-path order. State in darnengine/doc.go that a path touches every tree it lies in.
    origin: handler
  - id: F3
    severity: LOW
    class: design
    location: internal/impactset/guards.go:126
    summary: ModuleGuardTests silently skips a listed package whose directory is not under moduleDir, and filepath.Rel cannot fail for two absolute paths, so a path-spelling mismatch would drop every guard with no error
    fix: In ModuleGuardTests, replace the continue on the Rel error and the ".." prefix with a returned plain error naming moduleDir and the directory, or delete the branch and the Rel error check, since go list ./... at a module root lists only packages inside it.
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: scope
    location: internal/impactset/derive_test.go:43
    summary: No test row has the same guard name in two outside packages, so removing slices.Compact in buildCommand breaks nothing though the Shared Decisions require sorted deduplicated names
    fix: Add a TestCommandFor row, with the guards fixture extended or a row-local guards map, where two outside packages each hold TestGuardX, expecting the guard step to list TestGuardX once in the -run pattern.
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: scope
    location: internal/gatecli/run.go:149
    summary: The guardDir == "." branch, which passes the module root package as ., has no scenario row
    fix: Add a TestGateTest_Scenario guard row over a nested module whose root package holds a marked test, expecting a recorded stand-in invocation of . with its -run pattern.
    origin: lens:code-tests
  - id: F6
    severity: LOW
    class: consistency
    location: internal/loomshed/verifygate.go:30
    summary: NewVerifyGate's doc still says With wholeDiff false the gate takes the round form, naming the removed parameter
    fix: Change the sentence to With treeScope nil the gate takes the round form, matching the With treeScope non-nil paragraph below it.
    origin: lens:code-affected-set
  - id: F7
    severity: LOW
    class: consistency
    location: internal/darnengine/doc.go:3
    summary: The opening key list still defines verify as the command the darn verify gate, Publish and Finalize run, and omits tree_scope and tree_scope_verify, contradicting the restated paragraph below it
    fix: Restate the verify clause in the key list as the hub's whole verify command, with the same wording as the paragraph that begins verify is the hub's whole verify command, and add tree_scope and tree_scope_verify to the list with a pointer to the tree scope paragraph.
    origin: lens:code-affected-set
---

Cards 1 to 9 are implemented, and the plan's package list passes `lyx gate test --tags integration` for darnengine, impactset, gatecli, landingshed and configcli.
No `Pattern-Departure:` trailer exists, and the constraints carry no declared PATTERN changes.
No BLOCKING defect was found.

### [MEDIUM] wireDarn tells the verify gate the per-call scoped command as its told command

**Location:** internal/loomcli/darnwiring.go:263
**Issue:** Card 7 sets `c.env.VerifyCommand` to a closure over `darnengine.VerifyCommand`, so the gate's told command, the site's `BaseCommand` and step 1 of the darn form all stay the hub's whole `verify`. The diff left `darnVerifySource(location).command`, which Card 8 changed to resolve per call, so for a tree-scoped diff the told command and `BaseCommand` (`verifygate.go:174`) are the scoped command. `PATTERN-verified-tree.md`, which this diff restates, says `BaseCommand` is `darn.yaml`'s `verify`, so code and pattern text disagree. The classification also runs twice per arrival, once through the told command and once through `treeScope`. The assertion Card 7 asks of `TestWire_DarnRecipe`, that `c.env.VerifyCommand()` returns the hub's `verify` with tree scope on, is absent, so no test catches it.

### [MEDIUM] Nested trees drop the more specific tree's target

**Location:** internal/darnengine/config.go:321
**Issue:** `LoadConfig` accepts nested tree keys such as `docs` and `docs/api`. `TreeScopedCommand` assigns a path to the first tree in sorted order that holds it, so a path under `docs/api` counts only for `docs`, and the target of `docs/api` is left out of `{targets}`. A diff wholly under the hub's listed trees then runs fewer tests than the hub configured, and Merge-In and Publish skip the rest. No `TestTreeScopedCommand` row has nested trees.

### [LOW] ModuleGuardTests silently skips packages outside the module directory

**Location:** internal/impactset/guards.go:126
**Issue:** `filepath.Rel` of two absolute paths cannot fail, and `go list ./...` at a module root lists no package outside it, so the branch handles a state the code's guarantees rule out. If a spelling mismatch ever put every listed directory outside `absolute`, the scan would return an empty map with no error and `--guards` would run no guard.

### [LOW] Guard-name deduplication in buildCommand is unpinned

**Location:** internal/impactset/derive_test.go:43
**Issue:** The guards fixture holds only distinct names (`TestGuardA`, `TestGuardD`, `TestGuardD2`), so dropping `slices.Compact(names)` at `derive.go:217` breaks no test.

### [LOW] The root-package guard branch has no coverage

**Location:** internal/gatecli/run.go:149
**Issue:** Every guard package in the `TestGateTest_Scenario` modules sits in a subdirectory, so the `.` spelling for a root-package guard is untested.

### [LOW] Stale wholeDiff sentence in NewVerifyGate's doc

**Location:** internal/loomshed/verifygate.go:30
**Issue:** The parameter is now `treeScope`, so the sentence names a removed flag, beside a restated paragraph that says "With treeScope non-nil".

### [LOW] darnengine/doc.go opening sentence contradicts the restated verify description

**Location:** internal/darnengine/doc.go:3
**Issue:** Card 2 restates the doc's sentences on `verify`. The key list in the opening paragraph still says `verify` is what "the darn verify gate, Publish and Finalize run", which is false once Merge-In and Finalize run the scoped command for a tree-scoped diff and the gates run a derived one.

## Rejected

- Whitespace-only `tree_scope_verify` has no test row (lens:code-tests): the empty-means-off contract is pinned for `""`, and the `TrimSpace` calls are consistent across `normalizeTreeScope`, `TreeScopedCommand` and `darnTreeScope`.
- `darnReviewBudget` does not return its load error through `WithEditorWayForward` (lens:code-pattern-fit): it only logs a warning and falls back to a default, so it never refuses.
- `ModuleGuardTests` runs `go` from `PATH` rather than `request.goBinary` (lens:code-correctness): harmless, since only tests swap that binary.
- Refusals follow sorted key order, not the card's grouped order (lens:code-correctness): only which refusal appears first changes, not whether a bad config loads.
