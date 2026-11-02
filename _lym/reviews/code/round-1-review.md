---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-affected-set
    location: internal/websterengine/merriambase.go:43
    summary: Merriam's start base reads CLAUDE.md and CLAUDE.local.md at geom.WorktreeRoot, so in an anchored hub it no longer counts the rendered CLAUDE.local.md that now sits at the anchor
    fix: In MerriamBase, join each instruction file name onto the worktree's anchor directory instead of geom.WorktreeRoot. First grep the geometry type that geom is an instance of for an anchor path field, such as AnchorPath, and use it. Where the geometry carries none, defer this finding to the run's parent and leave the code as it is. Update the doc comment at merriambase.go:38 to say anchor, and extend TestMerriamBase in merriambase_test.go with an anchored row.
  - id: F2
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/fabricengine/warpexclude_integration_test.go:50
    summary: No test pins that wiring seeds the anchored exclude lines; every wiring, Reconcile and AddLocal row uses Anchor ".", so wireJunctionSet passing a wrong anchor would fail no row
    fix: In TestSeedWarpExclude_SeedsWarpOnlyOnce, add a row on hubforge.Shape{Anchor "backend"} that wires a pair through the same path the existing wiring row uses and asserts each line "/backend/<path>" appears exactly once in the exclude file and no root line appears. Reuse requireWarpExcludeOnce or countLine rather than a new helper.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/heddlecli/cli.go:52
    summary: The lym heddle help text and the heddlecli package doc still say file targets render at a worktree's root
    fix: In internal/heddlecli/cli.go at line 52 change "worktree's root" to "worktree's anchor directory". In internal/heddlecli/doc.go at line 28 change "write into a worktree's root" to "write into a worktree's anchor directory".
  - id: F4
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/heddlecli/verbs.go:335
    summary: Every lym heddle render and show case in cli_integration_test.go uses Anchor ".", so reverting verbs.go:295 or :335 to the worktree root would pass
    fix: In internal/heddlecli/cli_integration_test.go add one case on hubforge.Shape{Anchor "backend"} that runs lym heddle render and asserts both file targets exist under the anchor and not at the worktree root. Add a show case only if it shares the same setup.
---

### [MEDIUM] Merriam's start base misses the rendered file at the anchor

**Location:** `internal/websterengine/merriambase.go:43`
**Issue:** `MerriamBase` counts the instruction files `claudeengine.InstructionFiles()` names, `CLAUDE.md` and `CLAUDE.local.md`, at `geom.WorktreeRoot`.
Before the diff, heddle's `CLAUDE.local.md` sat at that root and was counted.
In a hub with an anchor it now sits at the anchor and is no longer counted, so the diff changes this consumer's behavior without touching it.
The repository's own `CLAUDE.md` already sat at the anchor and was missed, so the gap is older, but the rendered file joins it through this change.
The decision record lists no Merriam change, so the fix may belong to the parent; the fix text says when to defer.

### [MEDIUM] Wiring's anchored exclude lines have no test

**Location:** `internal/fabricengine/warpexclude_integration_test.go:50`
**Issue:** The diff changes the `wireJunctionSet` call at `junction.go:148` to pass `l.AnchorRel`.
The only anchored row, `anchored_hub_seeds_the_lines_under_the_anchor`, calls `SeedWarpExclude` directly.
The rows that wire, re-wire and add a pair all use `Anchor: "."`, where the root and the anchor give the same lines.
A wrong anchor at the wiring call would leave the file at the anchor untracked in git and fail no test.

### [LOW] Help text and package doc still say root

**Location:** `internal/heddlecli/cli.go:52`
**Issue:** The `lym heddle` long help says file targets render at "a worktree's root", and `internal/heddlecli/doc.go:28` says "later renders write into a worktree's root".
The diff updated the `render` verb's help in `verbs.go` and the engine docs, so these two contradict them.

### [LOW] The render and show verbs have no anchored test

**Location:** `internal/heddlecli/verbs.go:335`
**Issue:** `verbs.go` now passes `event.Anchor` to `Render` and `AnchorPath()` to `Show`.
Every `cli_integration_test.go` case runs with `Anchor: "."`, so reverting either call to the worktree root passes.
The engine-level anchored tests do not go through the verb.

## Rejected

- Lens code-correctness F-A, an old marked `CLAUDE.local.md` or `AGENTS.override.md` left at the git root of an already-rendered anchored worktree: the decision record asks for no migration or cleanup, and its Goal says the hub holds its heddle migration until the fix lands, so no root file exists to leave behind.
- Lens code-tests item 6, no test of the `filepath.Rel` error in `Seam.TookIn`: `EventFor` guarantees the anchor lies under the root, so the branch is caller misuse that the repository's trust-the-caller rule exempts from tests.
- Lens code-tests item 12, overlap between the seam `Anchored` rows and the integration test: the integration test adds the real `EventFor` and the git-untracked check, so it is not a duplicate.
