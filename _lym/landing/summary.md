# depends_on accepts a project or any entry that lands

A task can now name a project in `depends_on`, and dobby holds the task until that project is done.
Before, `depends_on` refused a project, and a member landing into a beam is not the beam landing into `main`.
A task that had to wait for a beam to reach `main` could not say so, and the hub orch watched the beam by hand.

## What changes

- `depends_on` accepts the slug of any entry that lands, a project included.
  The write that used to refuse a dependency on a project (`dependsOnProjectError` in `internal/boardengine/project.go`) is gone.
- A dependency is met when its entry is done, whatever its kind:
  - a task, or a beam, when its status is done, which a beam's landing into its parent sets;
  - a plain project when its status is done, or when it has a landing member and every landing member is done;
  - a slug no longer on the board, as before, was pruned as done.
- A landing member is a task or a project whose project key names the project.
  A note never lands, so it holds no project open.
- The README's After column and the Ready subsection treat a project dependency like a task one: a project not yet done is listed by its handle.
- `lym board set-deps` covers project dependencies in both checks.
  The cycle check follows a project to its landing members, so a member cannot wait on a task that waits on its own project.
  The overlap notice treats a project as the union of its landing members: a task waiting on a project and one waiting on a member of it wait on the same entry.
- Layers: a dependency on a project not yet done weighs as one on its deepest open landing member.
  A task waiting on a project therefore sits one layer below that member, and in Layer A when the project's open members are all running, ready or draft, or it has none.

## Design

- `internal/boardengine/dependency.go` holds the rule once, as `Dependencies`: `Pending`, `met`, `waitsOn`, `standsFor` and `overlap`.
  `internal/battencli/dobbyreads.go` (`DobbyTasks`), `ComputeLayers`, the README `afterCell`, the siblings check and the write-time cycle check all read it, so none keeps its own idea of done.
- The write-time cycle check in `store.go` now runs over `waitsOn`, with the depth-first walk extracted as `findCycle`.
  A cycle made of project membership alone is left to `checkProjectRules`, whose refusal names the project field to correct, instead of being reported as a dependency cycle.
- A membership loop stored by hand edit reads as not met, and `standsFor` stops on it, so neither recurses without end.
- `doc.go` in `internal/boardengine` describes the new rule in the dependency, Ready, After and Siblings passages.

## To check by hand

- A project that depends on an entry is still refused by the project rules; this change only lets a task depend on a project.
- The After-cell rendering of a project dependency in a README row, since a project dependency is judged by its landing members and not by its own status alone (`render.go`, `afterCell`, pinned in `render_test.go`).
- Recursion over project membership in `Dependencies.met` and `standsFor` when a stored membership loop exists (`dependency.go`, pinned in `dependency_test.go`).
- On a real hub: set a task's `depends_on` to a beam, run `lym board set-deps`, and confirm dobby holds the task until the beam lands into its parent.
  This end-to-end path was not run.
