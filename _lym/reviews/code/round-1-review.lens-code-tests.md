```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**
- Unversioned store reads `claude-file` as `agents-file`: `TestStore/StoreWithoutAVersionReadsClaudeFileAsAgentsFile`, rows `ClaudeFile` and `ClaudeFileBesideAPrompt`, through both `Load` and `Decode` (`store_test.go`).
- The migration drops a `codex-file` beside `claude-file`: row `ClaudeAndCodexFile`.
- `codex-file` alone is left as it is: row `CodexFileAlone`.
- A versioned store is left untouched: row `VersionedClaudeFile`.
- `Save` writes the version: the trailing Save-then-Load block in the same test. It would fail if `Save` omitted the version, because `Load` would migrate the target.
- `max(s.Version, storeVersion)` keeps a higher version: none.
- An unversioned entry already holding `agents-file` beside `claude-file` is deduplicated by the `!slices.Contains(targets, name)` branch in `upgradeStore`: none.
- The replayer decodes each side with `Decode`: none. `replayer_test.go` builds its entries with `entryFor("claude-file")` through a versioned save, so no unversioned replay is exercised.
- `claude-file` renders into `CLAUDE.local.md` only: `TestRender/ClaudeFileJoinsEntriesInOrderIntoItsFileAlone`, and `OnlyLimitsTheTargets/ClaudeFileAlone`.
- `codex-file` renders into `AGENTS.override.md` only: `OnlyLimitsTheTargets/CodexFileAlone`, and the `codex text` line in `AgentsFileRendersIntoBothFiles`.
- `agents-file` renders into both files, with the `@AGENTS.md` import and the AGENTS.md opening: `AgentsFileRendersIntoBothFiles`.
- `Only` with a shared target name skips it, so `Only: ["agents-file"]` renders nothing: none. This edge was changed by the `len(RendersInto) > 0` skip in the `Render` loop.
- `Show` refuses `agents-file` with a way forward: the added assertion in `TestRender` (`render_test.go`, around line 648).
- The overview's `targetReach` states the files for `claude-file` and `agents-file`: `TestRenderOverview` (`claude-file` and `agents-file` sections).
- The overview's "It also carries" line is not asserted for `codex-file`: its section is not asserted, but the shared loop is the same code path, so that is no gap.
- `list` prints `file_targets`, and `set --help` lists each target's files: `TestHeddleCLI` (`cli_integration_test.go`), the `wantFileTargets` assertion and the two help lines.
- Seam re-render paths, with `.claude/CLAUDE.md` now not re-rendering `codex-file`: the changed row `ClaudeDirClaudeMd` in `TestSeam` (`seam_test.go`).
- `ValidateEntry` accepts `agents-file` through its `LookupTarget` check: no direct test. It is a one-line table lookup and the `agents-file` entries in the render tests use the same table.
- `show --help` and the `doc.go` prose are static text and exempt.

**Findings**
1. LOW, `scope`, `internal/heddleengine/replayer.go:71-83`. `Decode` migrates each replay side, but no test replays an unversioned `heddle.json`. A regression that went back to a raw decode in the replayer would pass every test. Evidence: `replayer_test.go:37-39` builds `claude-file` entries only through a versioned save. Fix: add one `replayer_test.go` case with unversioned base, parent and commit bytes that holds `claude-file`, and assert the replayed result reads as `agents-file`.
2. LOW, `scope`, `internal/heddleengine/store.go:129-155`. Two `upgradeStore` branches have no row: an unversioned entry already holding `agents-file` beside `claude-file` (the dedup branch), and a store with a version above `storeVersion` (kept by `max` in `MarshalJSON`). Evidence: the rows in `store_test.go` around lines 260-285 hold neither. Fix: add the two rows to `StoreWithoutAVersionReadsClaudeFileAsAgentsFile`, and a `Save` round-trip row for version 2.

**Test economy:** no duplicate or redundant new test found. The `list` and `set --help` assertions pin different surfaces and are generated from the table, not static prose.

**Tool calls:** none. The inherited context already held every file this lens needed.
