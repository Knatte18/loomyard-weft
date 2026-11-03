```yaml
lens: code-correctness
outcome: reported
```

Task: review the correctness of the changed code (lens: code-correctness).

**List: function or branch, case run, verdict**

- `Dependencies.met`, task and pruned slug: correct. An open task is unmet, a done task is met, and a missing slug is met.
- `Dependencies.met`, beam: correct. A beam whose members are all done but which has not landed is unmet.
- `Dependencies.met`, plain project with no members: correct. It stays unmet until marked done.
- `Dependencies.met`, plain project with an open or abandoned member: correct. It is unmet.
- `Dependencies.met`, nested plain project: correct. The recursion ends when the chain is acyclic.
- `Dependencies.met`, project whose membership chain loops, for example `A.project = B`, `B.project = A`, or a project naming itself: incorrect, see finding 1. It recurses without end.
- `NewDependencies`, note member: correct. Notes are not indexed as landing members.
- `Pending`, task with empty `depends_on`: correct. It returns nil.
- `waitsOn` and `standsFor`, loop-free input: correct.
- `standsFor`, project whose membership loops: incorrect, same cause as finding 1.
- `overlap`, X and Y on the same project, X on a project and Y on its open member, and X on a member and Y on the project: correct.
- `overlap`, project whose only other member is done: correct. A done member does not link the two tasks, as the third siblings test row shows.
- `addedSiblings`, project dependency that is met: correct. It is skipped before `added`.
- `ComputeLayers`, `detectCycleDFS` over `waitsOn`, with the project and its member linked through an outsider: correct. It reports a cycle.
- `ComputeLayers`, `getDepth` project branch:
  - Project with no open member: correct. Depth 0, so the dependent lands in Layer A.
  - Project with a deep open member: correct. The dependent sits one layer below that member.
  - Beam held by a run: behaviour change, see finding 2.
- `ComputeLayers`, membership loop in stored data: `met` overflows the stack before `detectCycleDFS` can report, see finding 1.
- `boardView.afterCell`: correct. A met dependency is dropped and a project that is not done shows its handle.
- `DobbyTasks`, `Pending`: correct. A pruned slug is not pending, and the result is nil when nothing is pending.
- `validateWrite`, project built from `projected`: correct. A replaced entry is substituted and a new one is appended.
- `validateWrite`, member depending on its own project, and an outsider waiting on the project while a member waits on the outsider: correct. Both are refused as a cycle.
- `validateWrite`, cycle of project membership alone: correct. It is left to `checkProjectRules`, which `board.go:143` runs in the same write critical section after the store mutation. The `replay.go:465` and `replay.go:528` paths also call `validateWrite`. I did not check whether they reach `checkProjectRules`.
- `findCycle`: correct on direct, mixed and self cycles, and it terminates.
- `checkProjectRules`, removed branch: correct. The remaining membership and order checks are unchanged.

**Findings**

1. LOW, `design`: `internal/boardengine/dependency.go:107-117` (`met`) and `:128-136` (`standsFor`) recurse over project membership with no visited set.
   - A stored loop, or a project naming itself as its own project, overflows the stack. `checkProjectField` refuses loops on write (`project.go:97-103`), so only already-stored or hand-edited boards reach it.
   - The overflow would hit `ComputeLayers` (`layer.go:74`) and every `lym board` render. It happens before `detectCycleDFS` can return its cycle error, and the old code had no such failure mode for membership.
   - A visited guard in `met` and `standsFor` would close it.

2. LOW, `design`: `internal/boardengine/layer.go:106-120`. The project branch of `getDepth` now runs before the `layerMap[slug] != ""` check, so a project a run holds (a beam in Webster, Running layer) weighs as its deepest open member instead of depth 0.
   - The `ComputeLayers` doc comment says a dependency on a running task counts as one on a ready task, and does not say this holds for a running beam.
   - Whether it is wrong depends on intent: the dependent is held anyway because the beam is unmet. Layer placement only.

3. LOW, `design`: `internal/boardengine/store.go:392-408`. The write-time cycle check follows every landing member of a project, including done ones, but `ComputeLayers` skips met entries (`layer.go:74`).
   - Example: done member M depends on T, and T is written to depend on P (M's project). The write is refused as a cycle although M is done and P no longer waits on M.
   - The old check also followed done tasks (`store.go:659`). The project expansion widens the effect: a dependent can now be refused because of a done member.
   - Not a deadlock, only an over-refusal.

4. LOW, `design`: `internal/boardengine/store.go:404-407`. The membership-only suppression walks `findCycle(membership, incoming.Slug)` from the incoming entry.
   - If the incoming project also reaches a separate stored membership-only loop, a real mixed cycle is reported as membership-only and not refused here.
   - It needs an already-stored loop, which the rules forbid.

**Tool calls**
1. Grep for `checkProjectRules` and `validateWrite(` call sites in `internal/boardengine`: my context did not hold where the project rules run relative to `validateWrite`.
