# Card 21 — fabric-drop-local

**Edit:**
- `internal/fabricengine/destroy.go`
- `cmd/lyx#destructiveGuardRecordingExecutors`
- `cmd/lyx#destructiveGuardMutatingResultTypes`

**Create:**
- `internal/fabricengine#DropLocalResult`
- `internal/fabricengine#DropLocal`
- `internal/fabricengine_test#TestDropLocal`

**Intent:**
The destruction primitive behind `lyx adopt --drop-local` (D5).
`DropLocal`, in `destroy.go` behind the shared check pipeline (`PATTERN-fabric-destruction-chokepoint`) and filling its caller's `*Mutations` record (`PATTERN-mutation-record`), keeps each of the pair's local branch tips origin lacks under a local `archive/<slug>/<tip>` tag, then moves the branch to origin's tip with `reset --keep` in its worktree, refusing as the reconcile's dirty state does, naming the paths, when a tracked change sits there; it pushes nothing; `DropLocalResult` embeds `MutationRecord` and lists the tags it kept.
`destroy.go`'s header names the new primitive, and `destructiveGuardRecordingExecutors` and `destructiveGuardMutatingResultTypes` register `DropLocal` and `DropLocalResult`.
`TestDropLocal` (integration) covers a records branch and a code branch ahead of origin tagged and moved to origin's tip with the mutations recorded, and a dirty tracked path refusing with nothing moved.

**ImpactSummary:** fabricengine gains one destruction primitive, and the destruction guard's tables gain one executor and one result type; no existing caller changes.

**Commit:** 21: fabric: drop a pair's local-only commits to origin's tips, archived
