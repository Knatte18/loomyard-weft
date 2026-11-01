```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test row that pins it, or none.

**Per-behaviour list**

- Root with open sub-projects gets a `No project` heading and meaning line at the sub-projects' level, with its entry sections at `level+2`. Pinned by `TestRenderReadmeGolden` in `render_test.go`. Row "each project renders to its own file under its parent" uses `noProjectHeading(3)` at line 611, row "on a board with no prime owner…" at line 801, and row "nested projects stop deepening…" at line 878.
- A non-root project with open sub-projects gets an `Own entries` heading and meaning line, and its entry sections move to `level+2`. Pinned by the same test. In the README, `ownEntriesHeading(4)` at line 528 covers a depth-3 project, and the deep row covers 4, 5 and 6 at lines 869–875.
- The project-file path (the header stands as `#`, sub-projects at `##`, `## Own entries`, entries at `###`) is pinned by `ownEntriesHeading(2)` at line 664, with Tasks at `###`.
- A project with no open sub-project renders unchanged, with entries at `level+1` and no own-entries heading. Pinned by the leaf-project rows that still use `emptyProjectEntries(n)`, and by the unchanged project-file rows. The test file shows no change to lines in the rows without sub-projects.
- The depth cap: the own-entries heading and its sections stay at level six without colliding. Pinned by the row at line 832, which asserts `ownEntriesHeading(6)` followed by `emptyEntriesAtLevelSix`, and `heading` caps at six.
- `No project` and `Own entries` carry no entry count. Pinned because the golden helpers emit no count.
- An open project under a done project is counted under the root, so the root gets `No project` when it has open sub-projects. The row "each project renders to its own file under its parent" is described as covering it, and its `noProjectHeading(3)` at line 611 is in the same golden. I did not read that fixture, so I take this from the test's doc comment (lines 272–276).
- Root with no open sub-projects renders unchanged. Rows at lines 294, 381 and 416 are not shown changed in the diff and are the single-board cases, so a regression would fail them. I did not read their fixtures.
- Meaning-line text for both headings. Pinned by the `noProjectHeading` and `ownEntriesHeading` helpers.

**Findings**

- LOW, `consistency`, `render_test.go:933-965`. `emptyEntriesAtLevelSix`, `noProjectHeading` and `ownEntriesHeading` duplicate the literal heading and meaning strings in `render.go:133-134`. This is the usual golden practice and is not a defect; I raise no finding.

No behaviour the diff adds or changes lacks a pinning row, so I have no findings.

**Tool calls**
1. `grep` over `render_test.go` for row names and the heading helpers: my context lacked the list of golden rows and which one uses each helper.
