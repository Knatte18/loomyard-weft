```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test row that pins it, or none.

**Behaviour → pinning test**

- **D1, a project file for every project, open or done:** `TestRenderReadmeGolden` row "each project renders to its own file under its parent". Its `projectFiles` check lists all five files, `project-old.md` among them.
- **D1, the manifest drops a project file once the project is gone:** `TestRenderToDiskManifestCleanup/ProjectRemoved`.
- **D2, README.md holds only root entries:** the same golden row. The `want` text has no member entries.
- **D2, project file header (H1, `Under:` link, brief, beam or grouping, owner):** the same row's `files` for `project-ship.md` and `project-inner.md`, and the lone-beam row's `project-lone.md`.
- **D2, up link to the parent's file:** `Under: [Board](README.md)` for a root-level parent and `Under: [P1 Ship it](project-ship.md)` for a nested one.
- **D3, Projects section (linked bullets with brief, no lead line, no member table or Standing column):** `want` and `files` in the same row. The retired-string loop also rejects `Standing` and `Groups of entries`.
- **D3, child order by the parent's order key, then handle:** `project-ship.md`, where P5 precedes P4 because of `Order: ["later"]`.
- **D3, README.md child order by handle:** P1 then P2 in the same row, but the input order is already by handle, so a broken sort would not fail it. It is weak, not a gap.
- **D3, a done project is a linked Done row:** `P3 | **[Old project](project-old.md)**`.
- **D3, a bullet with no brief:** `- [P5 Later](project-later.md)`.
- **D4, layers hub-wide:** T3 in `project-ship.md` stands in Layer A while waiting on root T7 (Ready) and on running T2.
- **D4, counts count only the file's own entries:** `## Tasks (3)`, `## Done (1)` and the README counts.
- **D5, After as handles joined by `<br>`:** `T2<br>T7` in the projects row.
- **D5, After leaves out a done dependency:** the first row, where `T2` shows `T1` only. The projects row's T3 lists the done `shipped` (T4) in `DependsOn` and omits it.
- **D5, single dependency:** `T1`, `T5` and `T3` in the first and third rows.
- **D6, research first, headed `Research`:** the first golden row, which runs Research, Bugs, Enhancements, Other with `readmeTypes` including `research`.
- **Draft subsection, own place and not under Ready:** none added or changed by this diff. The old projects row's Draft section (task `chore`, in project `tidy`) is gone, and `project-tidy.md` is not in `files`. The first row's `### Draft (1)` is root-level only.
- **Replay recognising `project-<slug>.md` (`isBoardOverview`, `localBoardEntries` filter):** see F2.
- **`rename_test.go` change:** pins the member rendering in `project-grouping-two.md` and the old project file's removal. This is licensed by the board entry, because the renamed entry now renders in a project file.

**Findings**

- F1, LOW, class `design`, `internal/boardengine/render_test.go` (`TestRenderReadmeGolden`, "each project renders to its own file under its parent"): no row pins the case where an unknown or empty-handle dependency shows as a code span.
  - `afterCell` has the branch `found && entry.Handle != ""`, with the default `codeSpan(dep)`.
  - The only unknown-slug dependency in the fixtures is `base` in the projects row, and it is present on the board.
  - D5 states "A dependency that names no board entry shows its slug as a code span." The doc.go and summary claims rest on this branch.
  - Fix: add a `DependsOn` slug that is not on the board to a golden row and pin its code span in the After cell.
- F2, MEDIUM, class `design`, `internal/fabricengine/replay.go:519-570` (`isBoardOverview`, `isBoardPath`, `localBoardEntries`): the diff adds no test or row that pins project-file handling.
  - `internal/fabricengine/*_test.go` has no `project-` fixture, and no fabricengine test file changed in the diff.
  - The summary claims `TestSetOwners/CompetingWriteLosesToOrigins` (`internal/boardengine/ownerset_integration_test.go:448`) covers it. That test is unchanged by the diff, and I did not check whether its fixture renders project files.
  - Even if it does, it asserts only the owner outcome, so a broken `isBoardOverview` (for example one that accepts `project-x/y.md` or a `project-.md` path) would not necessarily fail it.
  - The `slug != ""` and no-`/` conditions are untested.
  - Fix: add a replay test where a board commit touches `project-<slug>.md` beside `board.json`. It should assert that the replay stays by-entry with no entry change named, and that a nested path or `project-.md` is not an overview.
- F3, NIT, class `consistency`, `internal/boardengine/render_test.go` (the `ProjectRemoved` subtest): it removes the only entry, so it covers deletion by emptying the board.
  - A project removed while other entries remain is exercised only through the rename test.
  - Fix: none needed.

**Tool calls (2 of 6)**
1. Grep for `project-` in `internal/fabricengine/*_test.go` and for `CompetingWriteLosesToOrigins`. My context lacked any fabricengine test coverage of the replay change.
2. `sed` of `ownerset_integration_test.go:440-520`, plus a grep for tests of `isBoardPath`, `isBoardOverview` and `localBoardEntries`. I needed to see what the summary's claimed covering test asserts. The grep returned no test file.
