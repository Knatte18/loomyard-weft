```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set.** For each change the diff makes, I looked for what it forces outside the lines it touches. I found no findings.

Per change, the set I found:

1. **`hubWorkspaceFolders` signature (new `names` parameter) and the hub file now listing every pair.**
   - **Set:** the only caller is `regenerateHubWorkspace` (`hubworkspace.go:170`), and it is updated. No other test calls it directly.
   - **Assertions:**
     - The old "hub file excludes beam members" assertions are in `hubworkspace_integration_test.go`, and the diff rewrites both.
     - A search for the old test-name phrases ("KeepsItsMembersOut", "OutOfTheHubFile") finds nothing left.
     - `verbs_regenerate_integration_test.go` also uses `requireFolderNames`, but it checks no beam-member exclusion.
   - **Covered.**
2. **New seam `InstallPairFolderNamer` / `PairFolderNamer`.**
   - **Set:** the install in `cmd/lym/main.go:165`, the assertion added in `prerunseams_test.go:55`, the fabric test that installs a fake namer (`hubworkspace_integration_test.go:172`), and the `fabricengine/doc.go` text at line 1665.
   - **Boundary:** `PATTERN-fabric-boundary` ("Fabric isolation") asks for a general seam. The seam takes only a hub path and returns slug-to-name, and fabric gains no board import. **Covered.**
3. **`boardengine.WorkspaceFolderNames`.**
   - **Set:** the `boardengine/doc.go` addition, the `docs/overview.md` ide bullet and `TestWorkspaceFolderNames`.
   - **Reuse:** it reuses `handleTitleSeparator` and `ReadTasks`. **Covered.**
4. **Doc text describing the hub file's listing.**
   - **Updated:** `fabricengine/doc.go`, `docs/overview.md` and the `RegenerateHubWorkspace` and `hubWorkspaceFolders` doc comments.
   - **Checked, still accurate:** `ideengine/spawn.go:42-47` says only that Spawn launches the hub file for the prime. `PATTERN-cwd-resolution.md:53` names the workspace files and says nothing about their contents. `SANDBOX-FABRIC-SUITE.md` mentions the file only in unrelated scenarios.
   - **Covered.**
5. **PATTERN declarations.** The constraints section lists no declared PATTERN changes, and the diff touches no `PATTERN.md` or `pattern/` file. Nothing is owed.
6. **Commit messages and departures.** Neither commit carries a `Pattern-Departure:` trailer, and none is needed.
7. **Stencils and fixtures.** The diff changes no stencil. The grep for "hub workspace" and "under a beam" in stencils and contracts matched only unrelated project-record directives.

Tool calls:
1. grep for `hubWorkspaceFolders` callers and for docs mentioning hub workspace listing; my context lacked repo-wide doc and caller references.
2. grep for the namer and seam symbols, and for files naming `code-workspace`; my context lacked other consumers.
3. Read `spawn.go` lines 40-48 and grepped the verbs-regenerate and hub-workspace tests, `PATTERN-cwd-resolution.md`, the sandbox suite doc and `fabricengine/doc.go`; my context lacked their content.
4. grep for stale test names and phrases and for other users of the folder-name helpers; my context lacked other tests asserting the old behavior.
