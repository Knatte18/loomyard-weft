```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test row that pins it, or say none.

**Per behaviour**

1. The hub file lists every pair at any depth, in slug order.
   - Pinned by `WritesABeamFileAndListsItsMembersInTheHubFileToo`, which expects `prime, Bolt, b, m`.
   - Pinned by `ListsSubBeamsInTheirParentBeamsFileByThemselvesOnlyAndEveryPairInTheHubFile`, which expects `b, c, m, n` and so covers nesting to depth 3.
   - A regression to "only the prime's pairs" fails both.
2. The prime and Bolt stay first and second. Pinned by the same two rows and by `assertFolders`, whose index-0 and `Bolt` path checks are unchanged.
3. Beam files are unchanged and name pairs by slug. Pinned by `NamesHubFileFoldersByTheInstalledNamer`, where the beam file for `b` must read `b, prime, Bolt, m` while the hub file shows `P1: m`. The two beam rows above also assert the beam file contents.
4. The installed namer renames a pair's hub folder.
   - Pinned by `NamesHubFileFoldersByTheInstalledNamer`, where `m` becomes `P1: m`.
   - A `""` value keeps the slug (`plain`).
   - A key naming no pair (`gone`) adds no folder.
5. The namer receives the hub path. Pinned by `namedHub != h.Path` in the same row.
6. With no namer installed, pairs are named by slug. Pinned indirectly by every other regeneration row, which expects bare slugs.
7. `WorkspaceFolderNames` returns the `&lt;P#&gt;: &lt;slug&gt;` name from the entry's own project.
   - `TestWorkspaceFolderNames`, row `EntriesInAProjectTakeItsHandle`, covers a task in a plain project, a task in a beam project, a project in a beam, and a task in a nested project (`deep` gets `P6`, not the beam's `P2`).
   - It covers a task with no project (`adopt` is left out).
   - It covers a task whose project is missing (`orphan`).
   - The project entries themselves, `bm` and `proj`, are not named, which the exact-map comparison asserts.
8. An unreadable board gives nil. Pinned by row `AnUnreadableBoardNamesNoEntry`, which passes truncated JSON.
9. No `board.json` gives no names. Row `NoBoardNamesNoEntry` passes under `maps.Equal`, which treats nil and an empty map as equal. It cannot tell "nil" from "empty", but the behaviour is the same either way.
10. The root pre-run installs the namer. `TestPersistentPreRunE_InstallsSeams` now requires `fabricengine.InstallPairFolderNamer`. Installation is skipped under `testing.Testing()`, so the AST check is the only possible pin and is adequate.
11. Handles are read from the board at each regeneration, not cached.
    - Pinned only partly. `WorkspaceFolderNames` has no cache, and `TestWorkspaceFolderNames` calls it once per fresh hub.
    - Fabric calls the namer once per regeneration, but no row regenerates twice with a changed namer result.
    - A cache added later in `pairFolderNames` or the namer would pass every test. **Severity LOW, class design, no test.** Location: `internal/fabricengine/hubworkspace_integration_test.go`, subtest `NamesHubFileFoldersByTheInstalledNamer` (calls `regenerateHub` once). It would be covered by a second `regenerateHub` call with a changed return value inside that row.
12. The names are read before the hub workspace lock is taken, a documented invariant at `hubworkspace.go` (the comment above `names := pairFolderNames(...)`) and in `doc.go`.
    - No test pins it. The lock-ordering row does not call a namer.
    - **Severity LOW, class design, no test.** If this does not warrant a test, say so in the finding. A namer that calls `lock` would deadlock or fail the order check.
13. The project's handle is empty, so the task keeps its slug.
    - This is the `handle != ""` branch in `handles.go`, in `WorkspaceFolderNames`.
    - `TestWorkspaceFolderNames` has no row with a project entry whose `handle` is empty. `orphan` covers only a missing project.
    - **Severity LOW, class design, no test.** Location: `internal/boardengine/handles_test.go`. Fix: add one project entry with no handle and a task under it to the board fixture.

**Test economy**
- `TestWorkspaceFolderNames` is unique per the redundancy report. It overlaps nothing, since the only other title test, `TestTitleWithHandle`, uses a different function and rule (the nearest beam, not the entry's own project).
- The seam test's comment on `NamesHubFileFoldersByTheInstalledNamer` and the edits to the two renamed rows overlap slightly on "hub file lists nested pairs", but each pins a different shape (single level with names, depth 3). No duplication finding.
- No added assertion is on static prose.

**Tool calls**
1. `grep` of `requireFolderNames` and `assertFolders` in `internal/fabricengine/hubworkspace_integration_test.go`: the diff did not show the helpers' bodies, and I needed to know whether they compare ordered names.
