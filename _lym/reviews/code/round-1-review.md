---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: pattern/PATTERN-config-layers.md:59
    summary: The Strictness section's degrading-callers list omits ideconfig, which the diff adds as a LoadOrTemplate caller and pins in the configstrictness test.
    fix: In pattern/PATTERN-config-layers.md line 59, add `ideconfig` to the degrading-callers set after `hibernateconfig`, so the list matches configStrictnessDegradingSet in cmd/lym/configstrictness_test.go.
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:code-tests
    location: internal/ideengine/spawn_hub_integration_test.go
    summary: The beam placeholder for a task, resolved through taskBeam, and the files OpenTask passes to the launcher have no test; the template's empty task list never reaches either.
    fix: In the task checks of spawn_hub_integration_test.go (checkTaskSlugOpensBareFolder and checkWindowCommandFollowsALaunch), set the hub ide.yaml task list to a file named with the placeholder (setIDEConfig), write that file under the task's anchor for a task forked from the prime and for one forked from a beam, and assert the launcher records the resolved path for Spawn and for OpenTask.
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:code-tests
    location: internal/ideengine/spawn.go:89
    summary: No ideengine test asserts that an ide.yaml failing to load fails Spawn, Seed and OpenTask, or that seed writes nothing in that case.
    fix: Add a case to the existing spawn integration checks in internal/ideengine/spawn_hub_integration_test.go that writes an invalid ide.yaml (an absolute entry) with setIDEConfig, then asserts Spawn, Seed and OpenTask each return an error wrapping the load failure, the launcher is not called, and no settings.json is written by Seed.
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: lens:code-tests
    location: internal/ideengine/spawn.go:108
    summary: The prime path, including the restructured seed branch and the prime open_files list, has no test.
    fix: In the prime check of spawn_hub_integration_test.go, set the hub ide.yaml prime list to an existing file and one naming the placeholder, and assert the launcher's recorded files for a prime Spawn equal the resolved paths in order; the same row covers the prime-name resolution of the placeholder.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/ideengine/spawn.go
    summary: filesToOpen branches for a directory entry and for a placeholder with no resolvable beam, and seed's unresolved-prime task path, have no row.
    fix: Add rows to the beam and task integration checks in spawn_hub_integration_test.go - an open_files entry naming a directory, which is skipped, and a placeholder entry for a task whose parent cannot be read, which is skipped; no new test function.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/vscode/config.go
    summary: applyMarkdownPreview asserts the workbench.editorAssociations value to a map, so the array form of that setting is replaced by a fresh map when render is on and the operator's entries are lost.
    fix: In applyMarkdownPreview in internal/vscode/config.go, when settings holds a workbench.editorAssociations value that is present and not a map, leave it unchanged and return in both the render-on and render-off paths, and add a row to TestWriteConfigOwnsMarkdownPreviewEntry with an array-form existing value asserting it is kept.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/ideengine/spawn.go:89
    summary: seed and OpenTask load ide.yaml over the caller's config stack, so a local override in the opened worktree is ignored when lym ide spawn runs from another worktree, although both keys are declared open to the local layer.
    fix: In seed and OpenTask in internal/ideengine/spawn.go, load ide.yaml with fabricengine.ConfigDirs(own), the opened worktree's own location, instead of ConfigDirs(l), and update the spawn doc comments that say l's config stack; keep placeWindow on l's stack.
---

### [MEDIUM] Degrading-callers list in PATTERN-config-layers not updated

**Location:** pattern/PATTERN-config-layers.md:59
**Issue:** The diff adds `internal/ideconfig` as a `configengine.LoadOrTemplate` caller and pins it in `cmd/lym/configstrictness_test.go`, but the rule document's Strictness section still lists the degrading callers without it. The repository's own rules must land in the same diff as the change they describe.

### [MEDIUM] Task placeholder resolution and OpenTask files uncovered

**Location:** internal/ideengine/spawn_hub_integration_test.go
**Issue:** `taskBeam` runs only when a task entry names the placeholder, and the template's task list is empty, so no test reaches it. `OpenTask` passing files to the launcher is likewise unasserted; the only `OpenTask` check pins the launcher's error and the window command. A regression dropping the files or resolving the wrong beam passes every test.

### [MEDIUM] Load failure of ide.yaml failing the verbs is not tested

**Location:** internal/ideengine/spawn.go:89
**Issue:** The diff moves the `ide.yaml` load before any write and makes a load failure fail `Spawn`, `Seed` and `OpenTask`. `ideconfig` pins the refusal in `LoadConfig`, but no `ideengine` test asserts the verbs fail and that `Seed` writes nothing, which is the behavior change the diff makes.

### [MEDIUM] Prime path uncovered

**Location:** internal/ideengine/spawn.go:108
**Issue:** The restructured `primeResolved` branch assigns the prime's files and no test sets a `prime` list or inspects the files a prime spawn passes to the launcher.

### [LOW] filesToOpen skip branches and the unresolved-prime path have no row

**Location:** internal/ideengine/spawn.go
**Issue:** A directory entry, a placeholder with no resolvable beam and the unresolved-prime task path are each skipped or taken by new code that no row exercises.

### [LOW] Array-form editor associations are replaced

**Location:** internal/vscode/config.go
**Issue:** `applyMarkdownPreview` reads the association value with a map assertion. An operator who holds the array form of the setting loses it when render is on, which contradicts the doc comment's promise to keep every other association.

### [LOW] ide.yaml loaded over the caller's stack, not the opened worktree's

**Location:** internal/ideengine/spawn.go:89
**Issue:** Both keys are declared open to a worktree's local override, but the load uses the caller's location. When the verb runs from another worktree, the target's override is inert. The summary states the caller's stack, but the declared local openness then has no effect for the worktree being opened.

## Rejected

- Launch argument order unpinned on Linux and Windows (lens:code-tests, LOW): a test would need an argument-builder helper extracted only for it, which the per-helper test economy rule forbids; the one-line append is identical on both platforms and read in the diff.
- render_markdown false threaded through seed has no engine-level row (lens:code-tests, LOW): the vscode table pins every outcome of the flag, and the engine passes one argument through; a row would duplicate that coverage.
- `lym ide open` on a beam or the prime resolves the placeholder to the parent (lens:code-correctness, LOW): the verb is documented for task worktrees and acts as the summary and doc comments say.
- A task forked from a non-beam task resolves the placeholder to that parent task's slug (lens:code-correctness, LOW): matches the doc comment and the summary's stated rule, "a task's parent's".
- The render_markdown key goes beyond the board entry's Goal (handler): the entry's principle that the window's opening is config and never hard-coded covers a switch for the rendering, and the off path leaves an operator's own association alone.
