```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I traced each function and branch the diff adds or changes and found no correctness defect.

**Each function or branch, the case I ran, and the result**

- **`shipped.json`, `lym-batching` and `lym-plan-edits-before-gate` retargeted to `agents-file`** (`internal/heddleengine/shipped.json:5-17`):
  - Ran an empty repository store at an anchor with no `AGENTS.md` or `CLAUDE.md`.
  - For `claude-file`, `reachesFileTarget("claude-file", ["agents-file"])` finds `agents-file`, and `targetTable` lists `claude-file` in its `RendersInto` (`target.go:66,105-115`). Both texts land in the shipped section under `shippedHeading` (`render.go:271-288`).
  - `codex-file` gets the same texts through the same path, with no opening file. `agents-file` is skipped as a target of its own (`render.go:112`).
  - Written comes out as `CLAUDE.local.md, AGENTS.override.md, .claude/settings.local.json`. Correct.
- **Override of a retargeted shipped slug** (`override.go:36-55`):
  - Ran `SetOverride("lym-batching", "tuned")`.
  - `validateObjectEntry` sees `["agents-file"]`, which is not a JSON-object target, so the override is accepted.
  - The override keeps the shipped targets (`override.go:13-14`), so the tuned text now reaches both files. Correct.
- **`Show(agents-file)`**: still refused as a shared target (`render.go:340-342`). Unchanged and correct.
- **`fileTargetWant`, `claudeFileWant`, `codexFileWant`** (`render_test.go:76-105`):
  - Ran `codexFileWant("repository rules", "codex text")`. It gives Marker, the opening, the heading, both shipped texts in order, the repository heading, then the text. This equals `fileText`'s output: production trims the opening file's `"\n"`, and the caller passes the opening already trimmed.
  - Shipped texts carry no leading or trailing newline, so the missing `Trim` in the helper is harmless.
  - Boundary case, an empty opening: the helper skips it, and so does production when the anchor holds neither file (`render.go:307-319`). Correct.
  - Edge, not reachable: with no shipped entry reaching the target, the helper still adds `shippedHeading` while production omits it. No caller hits that case under the real layer, so it is not a defect.
- **`TestRenderRemovesAStaleMarkedRenderEvenOffExcluded`** (`render_test.go:240-250`):
  - Ran an empty shipped layer, an empty store, `excluded` nil, and a marked `AGENTS.override.md`.
  - `claude-file` has no text and no file, so it is skipped. `codex-file` has no text and a marked file, so the file is removed and `Removed` is `[AGENTS.override.md]` (`render.go:131-138`).
  - `claude-settings` may hold derived entries; with no exclude list it gets a warning and goes to `Unwritten`, not `Removed`. `.heddle` is absent, so `removeTempDir` is a no-op. The assertion holds.
  - The `useShippedLayer` cleanup restores `shippedStore` (`render_test.go:177-179`).
  - Go resumes paused parallel top-level tests only after every sequential top-level test has finished and run its cleanups. `override_test.go:15-16` calls `t.Parallel()` before it reads `shippedStore`, so there is no race. Correct.
- **`TestRender/ClaudeFileJoinsEntriesInOrderIntoItsFileAlone`** (`render_test.go:261-277`): the codex file holds only the shipped section, since the store's `claude-file` entries do not reach it. The Written order follows `targetTable`. Correct.
- **`TestRender/BothFilesHoldTheShippedLayerWithNoEntry`** (`render_test.go:351-370`): both files are written with the shipped section, and Written lists all three files. Correct.
- **`AgentsFileRendersIntoBothFiles` and the `CodexFileOpensWithRootAgents` rows** (`render_test.go:279-349`): the expected text is ordered opening, shipped section, repository heading, entries. That matches `fileText`. Correct.
- **`TestSeam_PullTakeIn` prefix and suffix** (`seam_integration_test.go:93-97`): the rendered text is Marker, `new rules`, the shipped section, the repository heading, then `codex text`. It opens with the prefix and ends with the suffix. Correct.
- **`TestRenderOverview` assertion moved to the `agents-file` section** (`overview_test.go:71-80`): the shipped entries now name only `agents-file`, so their source line lands in that section. The tier-0 gate passed on this tree. Correct.
- **`TestStore`, the `ShippedEntriesArePinned` rows** (`store_test.go:184-185`): the rows match the new `shipped.json` targets. Correct.
- **`TestSeam` cases, unchanged**: these turn on `ReRender` paths and the exclude list, not on file content, so retargeting does not affect them (`seam_test.go:90-137`). Correct.

**Findings:** none.

**Tool calls**
- Read `internal/heddleengine/override.go` lines 1–80 and grepped `override_test.go` for `Parallel` and `shippedStore`. My context did not hold how `SetOverride` treats a shipped entry's targets, or whether `override_test` reads `shippedStore` before `t.Parallel()`.
