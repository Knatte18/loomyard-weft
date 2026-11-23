```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, which finds the test that pins each behaviour the diff adds or changes.

**Per behaviour, the test that pins it**

- **`validateWrite` drops the refusal of a note with `depends_on` (upsert, set-deps, demotion):** pinned by `TestKindDependencyRules` › "a note carries depends_on through upsert, set-deps and a demotion" (store_test.go:124–142), whose `GetTask` loop checks kind and deps.
- **Bolt replay lands a note with `depends_on`:** pinned by `TestFabricReplay` › `NoteWithDependsOnLands` (replay_test.go:482).
- **`set-deps` on a note stores the deps:** pinned by `TestWriteSiblings` › "note takes a dependency and makes no sibling" (siblings_test.go:227) and "set-deps on a note waiting on what an open task waits on" (siblings_test.go:297), both through `stored`.
- **`addedSiblings` skips a note as X (`x.Kind != KindTask`):** pinned by the two set-deps-on-a-note rows above. In each, Y is an open task sharing label `board` and waiting on D, so without the check a pair would be reported and `want` (nil) would fail.
- **`addedSiblings` skips a note as Y (`y.Kind != KindTask`):** pinned by "a note waiting on the dependency is no sibling" (siblings_test.go:302).
- **`addedSiblings` counts only task-held deps in the baseline (`beforeDeps` kind filter):** pinned by "promote of a note keeping a dependency an open sibling waits on" (siblings_test.go:292). Without the filter, D sits in X's baseline and the expected pair is not reported.
- **`promoteTo` keeps the note's deps, a given list replaces them, an empty list clears them:** pinned by `TestPromote` › `NoneGiven`, `Given` and `EmptyGiven` (store_test.go:1983–2006).
- **`Prune` strips a done slug from a note's `depends_on`:** pinned by `TestPrune` (store_test.go:2075), which wants `[live]`.
- **`BriefTask.Pending` from `Dependencies.Pending` (task and note, done dep left out, depends_on order):** pinned by `TestListTasksBriefLayerAndProposal` (store_test.go:1022–1042). It checks the Go field only; the `pending` JSON key and its `omitempty` are not asserted.
- **`RenderCompact` `after &lt;slugs&gt;` column, left out when no entry waits:** none. The only `--text` test is verbs_test.go:326, and it checks only the first line and each row's handle. Nothing in boardcli calls `RenderCompact` with a `Pending` entry.
- **README note-section After column (`boardView.noteTable`):** none. No boardengine test mentions an After column (grep of `internal/boardengine/*_test.go`), and the mutation survivors at render.go:682, 703 and 704 confirm it.
- **Help, doc and skill wording:** static prose, exempt.
- **Edited tests at the review base:** the removed refusal subtests ("a note with depends_on refused", "set-deps on a note refused", "note refused a dependency") and the replay row each pinned the refusal the decision record asks to remove. They are inverted to the new behaviour, not loosened, so no `Loosened-Check:` trailer is needed.

**Findings**

1. **MEDIUM**, class `design`.
   - Location: `internal/boardcli/text.go:111-113` (`RenderCompact` after column) and `text.go:121-128` (`afterText`).
   - Summary: no test pins the `after &lt;slugs&gt;` column that `list --text` gains, a behaviour the board entry asks for ("`lym board list --text` … show a note's dependencies").
   - What is missing: a row asserting a waiting task's and a waiting note's line ends with `after &lt;slug&gt;`, and that the column is absent when no entry waits.
   - Evidence: verbs_test.go:326–330 asserts only row handles; grep finds no `RenderCompact(` call in `internal/boardcli/*_test.go`.
2. **LOW**, class `design`.
   - Location: `internal/boardengine/render.go:697-710` (`boardView.noteTable`) and render.go:682 (its call in `entrySections`).
   - Summary: no test pins the README note table's After column, which appears only when a note in the section waits and holds the unmet dependency's handle. The mutation survivors at render.go:682, 703 and 704 name these lines.
   - Weigh against the testing rule: README layout read only by humans gets no layout test. Keep the finding only if the handler judges the README's After column to be data that other code reads.
   - Evidence: grep for `After` in `internal/boardengine/*_test.go` finds no README assertion, and the mutation report lists all three lines as survivors.

**Tool calls**

1. Searched boardcli and boardengine tests for `after`, `RenderCompact` and `Pending` assertions: the inherited context held no test files.
2. Searched boardengine tests for an After-column assertion and orchengine for `RenderCompact` usage: needed to settle whether the README and compact columns are asserted anywhere.
3. Searched boardcli tests for `--text`: needed to find the existing compact-listing test and what it asserts.
