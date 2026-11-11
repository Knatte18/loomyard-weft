# Fixer-report, code round 1

## Changes

- **F1** (LOW), fixed differently, commit `8b64be72`: `TestWorkspaceFolderNames` in `internal/boardengine/handles_test.go` now has a handle-less entry and a task under it in its board fixture.
  The task is left out of the `EntriesInAProjectTakeItsHandle` want map, so dropping the `handle != ""` check in `WorkspaceFolderNames` names it `": under-nohandle"` and fails the test.
  - Proposed change not made: a `kind: project` entry with no handle.
    The gate failed with it: `ReadTasks` runs `assignHandles`, which gave that project the handle `P1`, so `under-nohandle` was named `P1: under-nohandle`.
    A project always holds a handle, so the branch cannot be reached that way.
  - Own change: the entry is `{"slug":"nohandle","kind":"note","labels":["removed-type"]}`, a note with no configured type, which keeps an empty handle.
    The task `under-nohandle` names it as its project.
    The `AnEntryWithNoHandleKeepsTheName` row of `TestTitleWithHandle` uses the same note shape.
  - Gate: `lym gate test ./internal/boardengine` passes.

## Disputed

None.

## Deferred

None.

## Rulings

None.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

Every fork returned a report; the lens reports sit beside this file as `round-1-fixer-report.lens-<lens>.md`.
