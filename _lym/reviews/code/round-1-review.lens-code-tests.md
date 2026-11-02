```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests. Per behaviour, the test that pins it, or none.**

1. **Render joins the temp dir, the target path and the `File` at `Anchor`.**
   - `render_test.go`, `TestRender`, pins it through `renderFixture.anchor`.
   - `TestRender_WritesAtTheAnchorAndLeavesNoUntrackedFile` (`render_integration_test.go`, `Anchored`) pins it against a real hub.
   - That test asserts the files are absent at the worktree root.
   - Pinned.

2. **The `claude-file` import check reads `CLAUDE.md` and `.claude/CLAUDE.md` at the anchor.**
   - The `TestRender` import rows write their files with `writeAnchor` (`render_test.go` around lines 260–270).
   - `RenderRequest` carries no root any more, so the code cannot fall back to one.
   - Pinned. No test puts a `CLAUDE.md` above the anchor that must be ignored, but with no root in the request that case is unreachable in `Render`.

3. **The `codex-file` target opens with the anchor's `AGENTS.md`.**
   - `TestRender`, `CodexFileOpensWithRootAgents` (`render_test.go`), pins it.
   - `TestSeam`, `AnchoredAgentsMd` (`seam_test.go`), pins it with a separate root and anchor through the `strings.Contains("new rules")` check.
   - Pinned.

4. **`ReRender` paths are taken under the anchor, and a no-anchor worktree keeps its root-relative paths.**
   - Anchored: `AnchoredAgentsMd`, `AnchoredClaudeMd`, and `AgentsMdAboveTheAnchor` as the negative case.
   - No anchor: `AgentsMd`, `UnrelatedPath` and `ClaudeDirClaudeMd`, all with anchor ".".
   - Pinned.

5. **`Seam.Made` renders at the anchor, with and without an anchor.**
   - `TestSeam`, `MadeWritesBothFileTargetsAtTheAnchor`, rows `NoAnchor` and `Anchored`.
   - Pinned.

6. **`TookIn` returns an error when `filepath.Rel(ev.Root, ev.Anchor)` fails.**
   - No test covers it.
   - Misuse of the event by a caller; the code-quality rule says not to test this. Not a finding.

7. **`EventFor` carries `Anchor` as `l.AnchorPath()`.**
   - `TestRender_WritesAtTheAnchorAndLeavesNoUntrackedFile`, `Anchored`, builds the event with `EventFor` and renders from `event.Anchor`.
   - An empty or wrong `Anchor` would fail the file stat at `hub.Location.AnchorPath()`.
   - Pinned.

8. **`SeedWarpExclude` writes the `/<anchor>/<path>` lines and no root line.**
   - `anchored_hub_seeds_the_lines_under_the_anchor` (`warpexclude_integration_test.go:141`) pins it.
   - Pinned.

9. **A no-anchor hub keeps its `/<path>` lines.**
   - The existing rows (Anchor ".") and `warpExcludeLines`.
   - Pinned.

10. **Wiring seeds the anchored lines.**
    - The changed call is `wireJunctionSet` in `junction.go:148`, which passes `l.AnchorRel`.
    - Every wiring, `Reconcile` and `AddLocal` row in `warpexclude_integration_test.go` uses `Anchor: "."`. The only anchored row calls `SeedWarpExclude` directly.
    - If `wireJunctionSet` passed `"."` or the wrong location's anchor, no row would fail.
    - Severity: MEDIUM, class `design`.
    - Location: `internal/fabricengine/warpexclude_integration_test.go:50`, and `junction.go:148` for the untested call.
    - Fix: add an anchored row that wires a pair, or re-wires one, and asserts `/backend/<path>` appears once and the root line does not.

11. **The `lym heddle render` verb passes `event.Anchor`, and `heddle show` passes `AnchorPath()`.**
    - `cli_integration_test.go` runs every case with `Anchor: "."` (lines 217–623).
    - Anchor and root are identical there, so reverting `verbs.go:295` or `:335` to the worktree root would pass.
    - The engine-level anchored tests do not call the verb.
    - Severity: LOW, class `design`.
    - Location: `internal/heddlecli/verbs.go:295` and `:335`, with `cli_integration_test.go` as the missing coverage.
    - Fix: add one anchored case for `render`, and for `show` if it is cheap, asserting the file lands at the anchor.

12. **Test economy.**
    - The seam test fixtures and the integration test were converted to tables rather than duplicated.
    - Both `Anchored` subtests of `TestSeam` and the integration test overlap lightly in what they assert. The integration test adds the git-untracked check and the real `EventFor`, so I raise no finding.

**Tool calls**
1. One grep of `cli_integration_test.go` and `warpexclude_integration_test.go` for `Anchor` shapes, because the diff did not show whether the CLI and wiring rows use an anchored hub.
