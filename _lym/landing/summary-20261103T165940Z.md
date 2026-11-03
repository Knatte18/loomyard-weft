# depends_on accepts a project, and dobby holds the task until it is done

A task can now depend on a project as it depends on a task.
Before, the board refused any `depends_on` naming a project, so a task that had to wait for a beam to land in `main` was parked as a draft and watched by hand.

## What changes

- **The rule.** A dependency is met once its entry is done, whatever its kind.
  A task, or a beam, is done when its status is `done`; `lym beam land` sets that once the beam lands into its parent, so a member landing into the beam does not release the dependent task.
  A plain project is done when it is marked `done`, or when it has at least one landing member and every landing member is done.
  A landing member is a task or project under the project. A note never lands, so a note member never holds a project open.
  A slug no longer on the board still counts as pruned and done.
- **One judge.** The new `boardengine.Dependencies` (`internal/boardengine/dependency.go`) is where that rule lives.
  Dobby's start hold (`battencli.DobbyTasks`), the layers (`ComputeLayers`), the README's After column, the write's cycle check and the siblings notice all read it.
- **The write check.** The project refusal in `project.go` is gone.
  `validateWrite`'s cycle check now follows a project to its landing members.
  A member that depends on its own project is refused as a cycle, and so is a member that waits on a task that waits on the member's own project.
- **README.** A dependency on a project that is not done keeps the task out of Ready and shows in its After column by the project's handle.
  It weighs as a dependency on the project's deepest open member, so the task sits one layer below that member.
  It lands in Layer A when the project's open members are all running, ready or draft, or when it has no open member.
- **Siblings notice.** A dependency on a project counts as the project plus each landing member not yet done.
  So a task newly waiting on a project and a task waiting on one of its members are reported as siblings, and the other way round too.

## Design decisions

- A beam is met only by its own `done` status, never by its members, because the operator's case is the beam landing into `main`.
- A plain project with no landing member is not met until it is marked done.
  Otherwise a project created empty would release its dependents at once.
- An abandoned member is not done and keeps a plain project open. The operator closes such a project by marking it done.
- The siblings notice ignores a sibling's dependency that is already met, so a done member does not link two tasks.

## Tests

- `TestDependenciesPending` covers the rule for each kind: an open or done task, a pruned slug, a beam whose members are done but which has not landed, a landed beam, and the plain-project cases (all members done, one open member, no members, marked done, holding an unlanded beam).
- `TestProjectDependencies` replaces the old refusal test. It checks that upsert, set-deps and batch accept a project dependency, and it covers the two cycle refusals.
- `TestComputeLayers`, `TestWriteSiblings` and `TestDobbyTasks_ReadsDependenciesParentAndNotifyOnLand` gain rows for project dependencies.
