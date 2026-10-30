# Decision record — readme-omits-csharp-note

## Goal

Make the board README show every open project's members, so no entry leaves the README's view, and clear up the priority column name and the stray `grouping` line on project pages.

## Scope

- In: the README layout in `internal/boardengine/render.go`, its golden tests in `internal/boardengine/render_test.go`, and the README description in `internal/boardengine/doc.go` and `docs/overview.md`.
- In: the `P` column of the Notes tables renamed `Prio` on every board file.
- In: the `grouping` line under a project page's brief removed; a beam page keeps its `beam` line and owner.
- Out: the store, `lym board list` and any other view; project pages keep their Projects list of nested projects.

## Decisions

### D1 — One README section per open project, at the top

The README opens, after its header and intro, with one `##` section per open project on the board, nested projects included, so a nested project's members are visible from the README too.
The heading is the project's handle and title linked to its project page, followed by the number of members listed, as other README headings count theirs.
Under it come the project's brief, when it has one, and one table of its open members of every kind, in the README's entry order.
The table is laid out as a Notes type-group table: handle, `Prio`, the entry cell (bold title with brief, profile and run-setting bullets), the slug, `Age<br>Revised` and the labels; an empty project shows `_None._`.
Sections run depth-first: each root-level project in the root's order then by handle, each followed by its nested projects in its own order then by handle.
Done members stay out, as they do in the type groups; the project page's Done section lists them.
The README's old Projects bullet list is replaced by these sections.

### D2 — The prime renders as a project section after the others

The root project's own content, its Tasks, Notes and Done sections, sits under a `## Prime` heading below the project sections, with one line saying it holds the entries under no other project.
Its sections drop one heading level in the README (`### Tasks`, `#### Running`, `### Notes`, `#### Research`), so the markdown outline nests under Prime.
Project pages keep their sections at their present levels.

### D3 — The priority column is headed Prio

The Notes tables' priority column, and the new project-section tables, are headed `Prio`, so the column no longer shares the letter `P` with a project's handle.

### D4 — A project page drops the grouping line

A non-beam project page shows its brief and nothing in place of the `grouping` word; a beam page still shows `beam` and its owner.

## Constraints

- `PATTERN-test-speed`: tests run through `lym gate test`; the golden test in `internal/boardengine/render_test.go` is extended by rows and assertions, not new tests.
- Render stays a pure function in `internal/boardengine/render.go`, with section names and columns declared there alone.

## Auto-mode assumptions

- "Each project has its own section" covers nested projects too, since the trigger was a member hidden from the README.
- The prime's "section" is its existing Tasks, Notes and Done content under a Prime heading, not a second table of the same entries.
- Project sections count and list open members only.

## Open risks

- Demoting the prime's headings changes every README heading level, which any external reader keyed on `## Tasks` would notice; no Go code reads the rendered README.

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - open-project-under-done-project-has-no-readme-section (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- D1: a README golden row with root-level and nested projects shows one section per open project, depth-first in order then by handle, each with linked heading, member count, brief and a members table with the Prio column; an empty project shows `_None._`; no Projects bullet list remains in the README.
- D2: every README golden row shows `## Prime` after the project sections and the root's Tasks, Notes and Done one level deeper; project page goldens keep their levels.
- D3: every Notes table golden is headed `Prio`.
- D4: the non-beam project page goldens carry no `grouping` line, and the beam page still shows `beam` and its owner.
