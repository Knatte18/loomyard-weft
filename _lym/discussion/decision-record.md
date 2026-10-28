# Decision record — board-readme-per-project

## Goal

The board renders one markdown file per project instead of one README holding the whole hub, research notes lead the Notes section under the heading `Research`, and a task's After cell names its open dependencies by handle, one per line.

## Scope

- `internal/boardengine/render.go`: the README and per-project file rendering.
- `internal/boardengine/render_test.go`: the goldens and the file-set assertions.
- `internal/boardengine/doc.go`: the `# README` section describes the new layout.
- Out of scope: the board store, handles, `ComputeLayers` and `RenderOrder`, the design docs' own header, the `lym board list` and `get` views, and `board.yaml`'s keys.

## Decisions

### D1 — One file per project, named from its slug

Every project entry, open or done, renders to `project-<slug>.md` beside README.md, the prefix a constant in the renderer.
Rationale: a done project stays on the board while a member names it, so its members still need a file; the slug is unique and path-safe, and a fixed prefix adds no `board.yaml` key the hub would have to reconcile.
Bound: a removed project's file is dropped by the existing render manifest, so no stale file survives a render.

### D2 — README.md is the root project's file

README.md keeps its title, render time, prime owner and intro, then holds the projects with no parent project and every task and note whose project is empty.
A project file opens with the project's handle and title as its H1, the render time, a link up to its parent's file, the brief, `beam` or `grouping`, and a beam's effective owner, then holds the same sections as README.md for its own entries.
Rationale: the root project is implicit on the board, the empty project key, so its file is the one README.md the board already writes.

### D3 — Projects section lists the projects directly under the file's project

Each file opens its body with a `## Projects (n)` section listing the open projects whose project is this file's project, one bullet each: the handle and title linked to its file, then the brief.
The child projects follow the parent's order key, then by handle; on README.md, by handle.
The lead line "Groups of entries; a beam spawns its members from its own worktree." goes, and the section carries no lead line.
The per-project member table and its Standing column go, since a project's members now render in full in its own file.
A done project is a row of its parent file's Done table, its title linked to its file.
Rationale: the operator asked for linked projects with their short description, and most projects are groupings.

### D4 — Every other section is filtered by membership, layers stay hub-wide

Tasks, Notes and Done in a file hold only the entries whose project is that file's project.
Layers come from `ComputeLayers` over the whole board, so a task waiting on another project's task stands in the same layer it did.
Section counts count the file's own entries.
Rationale: the split is rendering only; handles and layers stay hub-wide.

### D5 — After names open dependencies by handle, one per line

The After cell lists each open dependency's handle, joined with a cell line break, in depends_on order.
A dependency that names no board entry shows its slug as a code span.
Rationale: a handle names one entry whichever file shows it, and one per line keeps a long list readable.

### D6 — Research notes first, headed Research

The Notes subsections run research first when `research` is a configured type, then the other types in board.yaml order, then Other.
The research subsection's heading is `Research`, not pluralised; every other type keeps the appended s.
Rationale: the operator asked for research where the audits stood, and "Researchs" is wrong.
Bound: a hub with no `research` type renders as before.

### Added after Discussion (parent, 2026-10-28): The records replay change in fabricengine is in scope

Decision: The change to internal/fabricengine/replay.go and its doc.go, which replays a board commit that touches a project-*.md file by entry like README.md, is part of this task's scope: the per-project files require it, since without it such a commit falls back to a git-level replay that conflicts on board.json. Keep it, with the tests review finding F1 asks for.

Rationale: Forced by the new file shape the task renders; refusing it would break records replay for every board write once project files exist.

## Constraints

- `Render` stays pure: entries in, file map out, no I/O.
- Section names, meaning lines and column headers stay declared in `render.go` alone.
- Tests stay tier0 goldens in `render_test.go`.
- `doc.go`'s `# README` section is updated in the same commit.

## Auto-mode assumptions

- The file prefix `project-` is fixed; no hub configures a design prefix that collides with it.
- A project file's H1 is its handle and title, matching the old H3.

## Open risks

- Links from outside the board to project sections of the old README break; the board README has no external anchors known to rely on them.

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - project-file-replay-untested (class design, severity MEDIUM, open in rounds 1)
  - replay-change-outside-record-scope (class decision, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- D1: a render of a board with an open and a done project returns `project-<slug>.md` for both, and RenderToDisk's manifest removes a project file once its project is gone.
- D2: README.md holds only entries with no project; a project file holds its header with the up link and its own entries.
- D3: README.md's Projects section lists only root-level open projects as linked bullets with briefs, a nested project appears in its parent's file, a done project is a linked Done row, and no lead line, member table or Standing column renders.
- D4: a task in a project waiting on a root task stands in Layer A in the project's file, and each file's heading counts count only its own entries.
- D5: an After cell shows handles joined by `<br>`, leaving out a done dependency.
- D6: with types bug, enhancement and research, Notes runs Research, Bugs, Enhancements, then Other.
