```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set — what the README render changes force outside the touched lines.

**Per change, the set found and whether it is covered:**
- README layout (project sections, `## Prime`, headings one level down): covered.
  - The consumers are `internal/boardengine/render.go`, `render_test.go`, `doc.go` and `docs/overview.md`.
  - My searches also hit `plugins/ly/skills/board/SKILL.md`, the root `README.md` and `internal/fabricengine`. Their README mentions either name the file only (fabricengine's `README.md` is the replay path match) or describe the board loosely. None names the removed `## Projects` section, a `## Tasks` heading or a `P` header.
  - No other test or fixture asserts README headings. In `internal/fabricengine/replay_integration_test.go:494` the only README check is that it contains the re-rendered entry `B2`.
- `P` column renamed `Prio`: covered.
  - The only remaining `columnPriority` reference is `render.go:361`.
  - A search for `| # | P |` outside `docs/research` found nothing. The matches there are unrelated bench tables.
  - `doc.go` and the render test doc comment now say `Prio`.
- `projectGroupingText` removed and the `grouping` line dropped: covered.
  - No Go reference to the constant remains.
  - The `grouping` hits elsewhere are unrelated. `internal/adoptcli` and `internal/boardcli` use "grouping project" as a concept, and `internal/boardengine/rename_test.go` uses `grouping` as a slug.
  - `doc.go` and the `fileHeader` comment were updated.
- `renderBoardFile` split into `renderReadme`, `renderProjectFile` and the `entrySections`, `table`, `childProjects` helpers: covered.
  - There is no remaining reference to `renderBoardFile`.
  - `Render` is the only caller and the diff updates it.
- PATTERN.md and `pattern/`: no entry applies and none is declared.
  - `PATTERN-test-speed` is met, because the golden test is extended by rows rather than new tests.
- Commit messages and `Pattern-Departure:`: neither commit carries a trailer and none is needed.
- Stencils: the diff touches none.

**Findings:** none.

**Tool calls (2):**
1. A grep across `.go`, `.md` and `.yaml` files for README headings, the `P` column and `Projects section`, to find consumers outside `render.go`, `render_test.go` and `doc.go`. My context had no repo-wide consumer search.
2. A grep for `Prime`, `grouping`, `README` and `Prio` in the board skill, stencils and `docs/overview.md`, plus a list of `Render(` callers in tests. My context lacked these.
