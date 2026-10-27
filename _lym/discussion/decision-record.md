# Board README lists draft tasks under Ready

## Goal

A draft task, one not yet released, no longer renders under Ready in the board README nor reads `Ready` as its `lym board list` layer, since dobby starts no draft task.

## Scope

- `internal/boardengine/layer.go`: `ComputeLayers` gives a draft task its own Draft layer, and `RenderOrder` orders it.
- `internal/boardengine/render.go`: the README Tasks section renders a Draft subsection, and a project member table names Draft as a draft member's standing.
- `internal/boardengine/doc.go`: the README part of the package doc states the Draft subsection.
- Tests in `internal/boardengine` for the layer and the rendered section.

Out of scope: the draft status rule, release, dobby's start rule and the README layout of projects and notes.

## Decisions

### A draft task gets its own Draft layer

Decision: `ComputeLayers` assigns every open task whose status is `draft` and that no run holds the layer `Draft`, ahead of the isolated and dependency-layer checks, so a draft task is never Ready, a letter layer or Independent.
Rationale: the layer says when a task can start, and a draft task starts on none of those until it is released; one layer keeps every unreleased task in one place, and its After column still names its open dependencies.

### A draft dependency weighs as a Ready one

Decision: a task depending on a draft task takes depth as on any special layer, so it lands in Layer A when that is its deepest dependency, and Layer A's meaning line names Draft beside Running and Ready.
Rationale: the dependent cannot start before the draft task is done, which is the same wait a Ready dependency imposes, and the existing special-layer depth rule gives it without a new branch.

### Draft renders last among the task subsections

Decision: the Draft subsection follows the dependency layers and Independent, is written only when it holds a task, and uses the same table as Ready and the layers.
Rationale: the Tasks section reads from what is closest to running to what is furthest, and an unreleased task is furthest; omitting it when empty matches Running.

### The `draft` code span on the title stays

Decision: a draft task's title cell keeps its `draft` code span.
Rationale: the cell is shared with project member tables and Running rows, where the span is the only draft mark.

## Constraints

- `PATTERN-test-speed`: tests run through `lym gate test`, in the lowest tier that fits, parallel by default.
- The layer stays computed at read time and is never stored.
- Docs land in the same commit as the code: `internal/boardengine/doc.go`'s README part.

## Auto-mode assumptions

- A draft task with open dependencies goes under Draft, not its letter layer: release is the first thing it waits on.
- An isolated draft task goes under Draft, not Independent, for the same reason.

## Open risks

- A reader of `lym board list` that matched on `Ready` to find startable tasks now skips draft tasks; no production Go code reads the layer field, so only operator habit is affected.

## Acceptance criteria

- `ComputeLayers` puts an open draft task with no open dependency in `Draft`, never `Ready`.
- A draft task with an open dependency, and an isolated draft task, are in `Draft`.
- A released task with no open dependency stays `Ready`, and a run-held task stays `Running` whatever its status.
- A task depending only on a draft task is in Layer A.
- The README renders a `### Draft (n)` subsection after the layers and Independent holding the draft tasks, omits it when no task is draft, and Ready counts no draft task.
- A project member table names a draft member's standing `Draft`.
