```yaml
lens: code-tests
outcome: reported
```

**Per-behaviour list**

| Behaviour | Pinning test | Verdict |
|---|---|---|
| Open task pending, done task met, pruned slug met | `TestDependenciesPending` rows `OpenTask`, `DoneTask`, `PrunedEntry` (`dependency_test.go:189-191`) | pinned |
| Beam met only by its own `done` status, not by landed members | rows `BeamWhoseMembersLandedWhileItHasNot`, `LandedBeam` | pinned |
| Plain project: met when every landing member is done, or marked done | rows `PlainProjectWhoseEveryLandingMemberIsDone`, `PlainProjectMarkedDone` | pinned |
| Plain project: open member holds it open | row `PlainProjectWithAnOpenMember` | pinned |
| Plain project: no members stays unmet | row `PlainProjectWithNoMember` | pinned |
| A note member holds no project open | `finished-note` fixture under `finished`, row `PlainProjectWhoseEveryLandingMemberIsDone` | pinned: counting the note would make `finished` pending |
| A project member is a landing member, and an unlanded beam holds the outer project open | row `PlainProjectHoldingABeamThatHasNotLanded` | pinned |
| `met` terminates on a membership loop, and so do `standsFor` and `getDepth` | none | **gap** (finding 1) |
| `DobbyTasks` pending list uses `Dependencies`, a beam dependency is pending, a pruned slug is not | `TestDobbyTasks_ReadsDependenciesParentAndNotifyOnLand`, `beam-dep` row (`dobbyreading_test.go`) | pinned |
| `DobbyTasks` with a plain-project dependency | none, but it goes through the same `Pending` as `TestDependenciesPending` | covered by that test |
| Layers: an open project dependency weighs as its deepest open member (A, B, Ready for a complete project, A for a beam with no open member) | `TestComputeLayers` first changed row (`layer_test.go`) | pinned |
| Layers: a cycle through a project and its member is refused | `TestComputeLayers`, "cycle through a project and its member" | pinned |
| `afterCell` hides a met project dependency and shows an unmet one by handle | none | **gap** (finding 2) |
| Removed refusal: upsert, set-deps and batch accept a project dependency | `TestProjectDependencies` `Accepted_*` rows (`project_test.go`) | pinned |
| Other write paths into `validateWrite` (merge, replay) accept it | none, but they share `validateWrite` | covered by the rows above |
| `validateWrite` cycle check follows a project to its members: member on own project, member on a task that waits on its project | `Refused_AMemberDependingOnItsOwnProject`, `Refused_AMemberWaitingOnATaskThatWaitsOnItsProject` | pinned |
| Plain `depends_on` cycle still refused with the new `findCycle` | none seen in this diff; the old cycle test is not in the diff | not verified (finding 3) |
| A membership-only cycle is left to `checkProjectRules` (second commit) | `ProjectChainWithACycle` (`project_test.go:155`) asserts "loops back" | pinned: if `membershipOnly` were dropped, the "cycle detected" message would replace it and the row would fail |
| Siblings: a task on a project and a task on its member, in both directions | the first two rows added in `siblings_test.go` | pinned |
| Siblings: a done member does not link two tasks | the third row | pinned |
| Siblings: a dependency on a project already met is not added | none | **gap** (finding 4) |
| Siblings: two projects sharing a nested member overlap | none | low-value gap, not raised separately |

**Findings**

1. MEDIUM, design — `internal/boardengine/dependency.go:107-117`, `:128-136`; `layer.go:106-120`.
   - No test covers termination of `met`, `standsFor` and `getDepth` on a project membership loop (A in B in A).
   - `checkProjectField` refuses new loops, but a stored loop still reaches `met`. `detectCycleDFS` calls `met(dep)` before it detects the cycle, so the recursion would not end.
   - The focus directive asks about termination, and no row in `dependency_test.go` or `layer_test.go` builds a loop.
   - Evidence: `dependency_test.go` fixtures hold only acyclic nesting (`outer` and `inner-beam`).

2. LOW, design — `internal/boardengine/render.go:590-604`.
   - The After column is a changed behaviour: it now drops a met project dependency and keeps an unmet one by handle.
   - The diff adds no row to `TestRenderReadmeGolden` or any render test.
   - The existing render fixtures name only task dependencies (`render_test.go:262-280`), and the second grep did not search `render_test.go` for a project dependency row.
   - The behaviour change sits only in the `b.dependencies.met(dep)` condition, so no render row would fail if it were reverted to `isDone`.

3. LOW, design — `internal/boardengine/store.go:386-408`, `:426-451`.
   - `validateWrite`'s cycle detection was rewritten as the new `findCycle`.
   - The diff touches no test for a plain task-to-task cycle. An existing test probably covers it, but I did not read one.
   - The only new cycle rows go through a project. The handler should confirm an existing row still reaches the "cycle detected" message for a plain `depends_on` loop.

4. LOW, design — `internal/boardengine/siblings.go:63`.
   - The `dependencies.met(dep)` filter now also drops an added dependency on a fully done plain project.
   - The third siblings row keeps `P` unmet through the open member `O`, so it never reaches that branch. Reverting `met` to `isDone` there would leave every siblings row passing.

**Tool calls**
1. One `grep` over `render_test.go`, `project_test.go` and `store_test.go` for After-column rows, project-loop refusals and project dependencies. My context lacked those files' unchanged parts. It confirmed `ProjectChainWithACycle` (`project_test.go:155`) and showed no project dependency in the After fixtures. It did not search `store_test.go` for a plain cycle, so finding 3 stays unverified.
