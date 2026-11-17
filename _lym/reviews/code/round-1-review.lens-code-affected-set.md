```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change in the diff, find what reads or asserts it outside the touched lines, and check whether the diff leaves any of it wrong.

**Per change, the set found, covered or not**

1. **`shipped.json`: `lym-batching` and `lym-plan-edits-before-gate` now target `agents-file` instead of `claude-file`.**
   - `store_test.go` `ShippedEntriesArePinned` — covered: both rows are updated.
   - `render_test.go` `claudeFileWant` and the new `codexFileWant` — covered. These are the only places the expected output is built.
   - `render_test.go` cases that expected no `AGENTS.override.md` (`ClaudeFileJoinsEntriesInOrderIntoItsFileAlone`, `ClaudeFileHoldsTheShippedLayerWithNoEntry`, `StaleMarkedRenderIsRemovedEvenOffExcluded`) — covered: the first is updated, the second renamed and updated, the third moved to a non-parallel test.
   - `CodexFileOpensWithRootAgents` and `AgentsFileRendersIntoBothFiles` — covered: their exact strings now go through `codexFileWant`.
   - `overview_test.go` `TestRenderOverview` — covered: the "shipped with lym" assertion moved to the `agents-file` section.
   - `seam_integration_test.go` `TestSeam_PullTakeIn` — covered: the exact match became a prefix and suffix check.
   - `seam_test.go` `TestSeam` — no change needed. It only checks which files are present, and its absent rows depend on which targets each path re-renders or on an empty exclude list, not on shipped content (`seam_test.go:99,113-120`).
   - `render_integration_test.go` — no change needed. It already expected both files written (`render_integration_test.go:54`).
   - `heddlecli/cli_integration_test.go` — no change needed. The refused-render rows expect no file whatever the content, and the write rows already expect both files (`cli_integration_test.go:723-779`).
   - `settings_test.go` non-parallel tests — no change needed. They swap in an empty shipped layer and check only the settings file.
   - No other Go file in `internal/` or `cmd/` reads `AGENTS.override.md` (grep returned nothing).
   - Bolt's `HEDDLE.md` still shows `Targets: \`claude-file\`` for `lym-batching` (`_bolt/HEDDLE.md:14`). It is outside this repository's diff, and `RenderOverview` rewrites it through the hub reconcile once per lym build, so it follows on its own. Not a finding.
   - `PATTERN-heddle-instructions`: lym's own instructions stay in the shipped layer and are rendered only by `heddleengine`. Compliant.

2. **`shipped.json`: the three lines that named Claude tools are reworded.**
   - The repository root `CLAUDE.local.md` still has the old wording (`CLAUDE.local.md:13,24,27`). The file is untracked (`git ls-files` printed nothing), and the board entry says it follows on the next render. Not a finding.
   - `internal/shuttleengine/claudeengine/batchingnudge.go:130` still says "every Edit and Write call together". It is a nudge only the Claude engine sends, so naming Claude's tools there is correct, and the board entry scopes the rewording to the shipped entries. Not a finding.
   - No spec or stencil under `_bolt/_lym/specs` quotes the shipped text.

3. **Test helpers: `fileTargetWant` added, and `claudeFileWant`'s parameter renamed from `imports` to `opening`.**
   - All call sites are in `render_test.go` and have been updated.

4. **Rubric checks on commit messages, departures and PATTERN changes.**
   - The one commit's message matches the diff and carries no `Pattern-Departure:` trailer.
   - The directive declares no PATTERN changes, and the diff touches no `PATTERN.md` or `pattern/` file.
   - The diff changes no stencil.

**Findings**

None. Every member of the affected set is either updated in the diff or changes on its own at the next render or reconcile.

**Tool calls**

1. Grep for `AGENTS.override` across `internal/` and `cmd/` outside the files already read, and for docs that describe the shipped entries' targets. The manifest only covered the heddle packages and fabricengine.
2. Grep for copies of the shipped headings and the old Claude-tool wording across the repo. The manifest had no search for duplicates of the text.
3. Grep of `_bolt` specs and `HEDDLE.md` for descriptions of the shipped entries. Those docs sit outside the repo and were not in the manifest.
