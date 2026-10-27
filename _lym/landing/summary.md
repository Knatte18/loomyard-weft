# Give draft tasks their own Draft layer instead of Ready

The board README listed an unreleased task under Ready ("waits on nothing open; can start now"), though dobby starts no draft task.
`ComputeLayers` handled done, running, project and isolated entries but not draft, so a draft task with no open dependency fell into `Ready`.
The same wrong layer showed in `lym board list`.

Closes #873

## What changes

- `ComputeLayers` puts every open draft task that no run holds in a new `Draft` layer, whatever its dependencies or isolation.
  A run-held task stays `Running`, a released task with no open dependency stays `Ready`.
- `RenderOrder` sorts `Draft` after Independent and before done.
- The README Tasks section renders a `### Draft (n)` subsection after the dependency layers and Independent, in the same table as the other subsections.
  It is omitted when no task is draft, as Running is.
- A project member table names a draft member's standing `Draft`.
- `internal/boardengine/doc.go` states the Draft subsection.

## Design decisions

- The Draft check runs ahead of the isolated and dependency checks, so a draft task with open dependencies, or an isolated one, is under Draft, not its letter layer or Independent.
  Release is the first thing it waits on, and its After column still names its open dependencies.
- A task depending on a draft task weighs it as a ready one, so it lands in Layer A when that is its deepest dependency.
  Layer A's meaning line now names Draft beside Running and Ready.
- A draft task's title cell keeps its `draft` code span, since the cell is shared with project member tables and Running rows, where the span is the only draft mark.
- A new `isDraft` helper in `draft.go` replaces the inline status check in `entryCell`.
- The layer stays computed at read time and is never stored.

## Check by hand

- Anything that matched `Ready` in `lym board list` to find startable tasks now skips draft tasks.
  No production Go code reads the layer field, so this affects operator habit only.
