# Board renders root as project P0 with its own file, last in the README

The board README used to open with a `## Prime` section that wrapped every project, and further down held root's own entries under a `### No project` heading.
"Prime" and "No project" named the same thing, and the `No project` heading had no file link, no facts line and no brief, unlike every project heading.

Root is now rendered as a project like the others:

- Root has the handle `P0` and the title `Root: entries under no project`.
  Board project handles start at `P1`, so `P0` never clashes with a board entry.
- Every write renders `project-root.md` beside the `project-<slug>.md` files.
  It opens like any project file, with its title, render time, a link up to the README, and a facts line.
  It then lists root's own tasks by layer, its notes by type and its done entries.
- The README keeps every table it showed before.
  Every open top-level project is now a `##` section, and root's section comes last as `## [P0 Root: entries under no project](project-root.md) (n)`.
  Its facts line names only the prime owner (`Owner \`neve\``), because root has no board entry to give it a slug, an age or a design link.
- The `## Prime` wrapper and the `No project` heading are gone, so every heading below them moves up one level.

## Design decisions

- Root's projects stand beside its section in the README, not under it.
  So root's section and `project-root.md` hold root's own entries only, and root's count in the heading leaves the top-level projects out.
- A project file with sub-projects keeps its `Own entries` heading.
  Root's layout does not duplicate it: that heading still marks where the last sub-project ends inside a project file.
- `project-root.md` links up to the README the way a top-level project's file does, so every project file has a way back to the overview.
- The README header still names the prime owner, which `TestPrimeOwnerReadAndRendered` reads.

## Open risks

- A board project with the slug `root` would render to the same file name as root's own file, and one would overwrite the other.
  No write rule reserves the slug yet.

## Tests and docs

- The README goldens in `internal/boardengine/render_test.go` follow the new layout.
  The projects row now also pins `project-root.md` in full and lists it among the project files.
  The nesting row gains a fifth project so it still reaches heading level six.
- `internal/boardengine/doc.go` and `docs/overview.md` describe root as `P0` with its own file, and its section last in the README.
