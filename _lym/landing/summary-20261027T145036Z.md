# Give draft tasks their own Draft layer instead of Ready

A task with status `draft` has not been released, and dobby starts no draft task.
The board still put a draft task with no open dependency under Ready ("waits on nothing open; can start now"), and `lym board list` reported its layer as `Ready` (issue #873).
`ComputeLayers` dealt with done, running, project and isolated entries but not draft ones, so a draft task fell through to depth 0.

## What changes

- `ComputeLayers` (`internal/boardengine/layer.go`) gives every open draft task that no run holds the new layer `Draft`.
  This comes after the done, running and project checks and before the isolation and depth checks, so a draft task is never Ready, a letter layer or Independent.
  For a task that depends on it, a draft task counts as a dependency at depth 0, the same as Running or Ready, so that task lands in Layer A when the draft task is its deepest dependency.
- `RenderOrder` puts the Draft bucket after Independent and before done entries.
- The README Tasks section (`internal/boardengine/render.go`) gets a `### Draft (n)` subsection after the dependency layers and Independent.
  It uses the same table as Ready and the layers, After column included, and is left out when no task is draft.
  Layer A's meaning line now reads "Waits only on Running, Ready or Draft entries."
  A project member table shows a draft member's standing as `Draft`.
- `isDraft` in `draft.go` is the one draft-status check that the layer code and the title cell share.
- The README part of the package doc (`internal/boardengine/doc.go`) describes the Draft subsection and its columns.

## Design decisions

- A draft task with open dependencies, and an isolated draft task, both go under Draft rather than their letter layer or Independent: release is the first thing either waits on, and the After column still names the open dependencies.
- The `draft` code span on the title cell stays.
  Now that every draft task renders under Draft, the span repeats what the section heading already says.
  The decision record gives a different reason for keeping it: that project member tables and Running rows rely on the span.
  That reason is wrong: member tables never showed the span, and a draft task is never Running.
  The span is kept anyway because it costs nothing and still marks a draft task wherever its cell is copied.

## Tests

- `TestComputeLayers` gets a row covering a draft task with no dependencies, one with an open dependency and one that is isolated, all of which land in `Draft`.
  The same row checks that a released task stays Ready and that a task depending on a draft task lands in Layer A.
- `TestRenderOrder` now puts a draft task between Independent and done entries.
- In `TestRenderReadmeGolden`, the draft task in the main fixture moves from Ready to a Draft section, and Ready then renders `_None._`.
  The project fixture's grouping member is now draft, which covers the `Draft` member standing and a Ready count that excludes it.
