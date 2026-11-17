```yaml
lens: code-tests
outcome: reported
```

**Per behaviour, the test that pins it**

1. `ListTasksBrief` prints the prime as `root` when the `ParentOf` seam answers `""` — `TestParentBeam/BoardCarriesTheParentBeamTheSeamAnswers` (`internal/boardengine/parentbeam_test.go:92`).
2. `ListTasksBrief` prints the prime as `root` with no seam (board's own entries) — `TestParentBeam/BoardWithoutASeamReadsItsOwnEntries` (`parentbeam_test.go:119`) and `TestListTasksBriefLayerAndProposal` (`internal/boardengine/store_test.go:1039`).
3. The list still prints the beam slug under a beam — `parentbeam_test.go:92` (`a`→`beam`), `store_test.go:1039` (`member`→`beamer`), `stepProjects` (`internal/boardcli/verbs_test.go:614`).
4. A seam error leaves the cell empty, never `root` (new switch ordering, `store.go:775-780`) — `TestParentBeam/AFailingSeamLeavesTheCellEmptyAndTheListSucceeds` (`parentbeam_test.go:150`; the seam returns `"beam"` with an error).
5. `get` envelope prints the prime as `root` (`outputGetTask`, `cli.go:1376-1379`) — `stepProjects` (`verbs_test.go:600`, `verbs_test.go:617`).
6. `get` envelope still prints the beam slug — `stepProjects` (`verbs_test.go:611`).
7. `get` with a nil task omits `parent_beam` — unchanged path; no changed row, not a new behaviour, no finding.
8. `Board.ParentBeamOf` keeps the empty slug for the prime — new assertion `parentbeam_test.go:98`.
9. `hubgeom.ParentOf` keeps the empty slug — code unchanged; not covered by a diff row; unchanged behaviour, no finding.
10. `list --text` shows a P0 root row (`RootProjectBrief` + `withRootProjectRow`), placed before the first note — `stepListAndFindText` (`verbs_test.go:347-354`), exact handle/kind/slug/title, blank dates and position.
11. `withRootProjectRow` places the row before the first **project** — none. Fixture holds only a task and a note (`verbs_test.go:342-343`), so a predicate matching only `KindNote` would pass.
12. `withRootProjectRow` appends the row at the end when there is no project or note (and on an empty board) — none; the `at < 0` branch (`text.go:95-97`) is unreached by any test.
13. `list --text --label` omits the P0 row (`cli.go:961-964`) — none. The only `--label` tests are JSON (`verbs_test.go:314`, `verbs_test.go:321`).
14. JSON `tasks` holds no root row — `stepListAndFindText`'s `--sort` rows (`verbs_test.go:369`, `verbs_test.go:372`) require the slugs to be exactly `[later soon]`/`[soon later]`.
15. `find --text` shows no root row — `stepListAndFindText` `wantFind` (`verbs_test.go:359-363`).

**Findings**

- **F1 — MEDIUM, design, `internal/boardcli/cli.go:961-964`** — `list --text --label` omitting the P0 row is a behaviour the diff adds with no test; dropping the `len(listLabels) == 0` guard would pass the suite. Evidence: `verbs_test.go:314,321` run `--label` only as JSON; no `--text --label` run exists.
- **F2 — LOW, design, `internal/boardcli/text.go:91-99`** — `withRootProjectRow`'s placement before a project, and its end-of-list branch (no project or note; empty board), are reached by no row; only the before-first-note case is pinned. Evidence: `stepListAndFindText` fixture `verbs_test.go:342-350` holds one task and one note.
- **F3 — LOW, consistency, `internal/boardcli/verbs_test.go:582`** — `stepProjects`' doc comment still says parent_beam "is empty under a grouping project", while the rows it documents now assert `boardengine.RootSubject` (`verbs_test.go:600-604`). Evidence: comment at `verbs_test.go:582` vs assertions `verbs_test.go:600-605`.

Tool calls:
1. Grep of boardcli tests for `--label` runs and of all tests for `withRootProjectRow`/`RootProjectBrief` — my context did not show whether any test runs `list --text --label` or reaches the root-row helpers directly.
