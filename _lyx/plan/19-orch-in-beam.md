# Card 19 — orch-in-beam

**Edit:**
- `internal/orchcli#Command`
- `internal/orchcli#refuseNonPrime` -> `func refuseNonPrime(worktreeName, primeName string, primeNameErr error, beam bool) error`
- `internal/orchcli#orchCLI`
- `internal/orchcli#orchCLI.startCmd`
- `internal/orchcli#TestRefuseNonPrime`
- `internal/orchcli#TestStart_Refusals`
- `internal/orchcli#refusalCase`
- `internal/orchcli#TestOrchIntegration_Refusals`
- `contracts/stencils/orch/orch-template-role.md`
- `internal/orchcli#reconcileHub`
- `internal/orchcli#resumeContextCommand`
- `internal/orchcli/doc.go`
- `docs/overview.md`

**Create:**
- `plan:internal/orchcli#TestOrchInBeam` -> `func TestOrchInBeam(t *testing.T)`
- `plan:internal/orchcli#refuseBeamStart` -> `func refuseBeamStart(slug, here, owner string, status *string) error`
- `plan:internal/orchcli#OrchLive` -> `func OrchLive(worktree *lyxcwd.Location) (bool, error)`
- `plan:internal/orchcli#orchLiveFrom` -> `func orchLiveFrom(st orchengine.State, strands func() ([]reedengine.StrandStatus, error)) (bool, error)`
- `plan:internal/orchcli#TestOrchLiveFrom` -> `func TestOrchLiveFrom(t *testing.T)`
- `plan:internal/orchcli#TestRefuseBeamStart` -> `func TestRefuseBeamStart(t *testing.T)`

**Uses:**
- `internal/hubgeom#ReedGeometry`
- `internal/orchcli#PrimePaths`
- `internal/hubforge#CopyHub`
- `internal/hubforge#AddPairWith`
- `internal/fabricengine#ReadOrigin`
- `internal/boardengine#ReadOwners`
- `internal/hubgeom#ParentOf`

**Intent:**
D8, orch: every `lyx orch` verb runs in the prime and in a beam, a pair whose origin record carries the beam mark (card 2), one orch per worktree.
The pre-run reads the acting worktree's origin record (`ReadOrigin`) after `RequireDrivableWorktree` and passes its mark to `refuseNonPrime`, which admits the prime and a beam and refuses every other worktree naming `lyx beam create <slug>` for a project or `lyx beam promote <slug>` for a task as the way to make it one.
Bound: what the wider guard lets through is an orch in a beam's worktree, whose files stay under that worktree's own `.lyx/orch/` and `_lyx/orch/` (the existing `orchPaths` over its location, PATTERN-durable-vs-ephemeral-state) and whose strand is `<shortname>:<slug>:orch`, the name reed already gives the orch role in a pair and the name `hubgeom.ResolveParent` gives a pair forked from that worktree; its watcher is its own.
`start` in a beam claims no prime and, through `refuseBeamStart` over the board's owners and the beam entry's status, refuses a beam another machine owns (effective owner by `hubgeom.ParentOf`), naming the owner and `lyx adopt <slug>`, and a beam whose entry is run-held by its landing (`IsRunStatus`), naming the landing and `lyx batten status <slug>`, so no orch starts between the landing's claim and its teardown.
The input that reaches the landing refusal is a `lyx orch start` in the beam after `lyx beam land` claimed it; `TestRefuseBeamStart` reaches each refusal and the pass, and `TestStart_Refusals` gains a row through the start harness with the seam set.
`OrchLive` is the engine liveness predicate `lyx orch status` uses, as a function other modules call: the worktree's orch state records a strand that reed tracks and reports live (`orchLiveFrom` over the loaded state and the worktree's reed strands); hubgeom's walk (card 18) and the ready-notice routing (card 26) take it.
`TestOrchLiveFrom` rows: no strand recorded, a tracked live strand, a tracked dead strand, an untracked strand and a strands error.
`TestRefuseNonPrime` gains rows: a beam passes, a non-beam pair is refused naming `lyx beam create` and `lyx beam promote`.
Both orch refusals, `refuseNonPrime`'s and the pre-run's wrap of the drivable-worktree refusal, open with "lyx orch runs from the hub's prime worktree or a beam" in place of "runs from the hub's prime worktree only", and `TestRefuseNonPrime`'s existing non-prime row asserts that opening.
The integration helper `refusalCase` runs for a task worktree and for the prime's records sibling alike, so it asserts that shared opening and no longer the "prime worktree only" wording; the beam verbs the task-worktree refusal names are asserted by `TestRefuseNonPrime`'s row, since the records sibling's refusal names none.
`refusalCase`'s doc, `TestOrchIntegration_Refusals`'s doc and `refusal_integration_test.go`'s file header, which describe a prime-only rule, are rewritten to describe the prime-or-beam rule.
`reconcileHub` resolves the prime's location from the acting location before it builds the teardown and its seams, since the hub-scope reconcile runs from the prime whichever orch starts it.
The resume-context hook changes directory to the acting orch's own anchor, the beam's in a beam: `resumeContextCommand`'s comment and the package doc's session-start hook section say so in place of "the prime's anchor".
`TestOrchInBeam` over a hubforge hub with a marked beam pair `b` and a recorded shortname: the orch paths of `b`'s location lie under `b`'s anchor `.lyx/orch/` and `_lyx/orch/`, never the prime's, and `ReedGeometry` of `b` names the orch role's strand `<shortname>:b:orch`.
The orch role stencil tells an orch to read and start only entries whose `parent_beam` is its own worktree, the prime's orch those with an empty one, and never to pass `--adopt-local`; the orch package doc, `Command`'s Long sentence "Every verb runs from the hub's prime worktree only" and `docs/overview.md`'s orch row, which say "prime only", are rewritten to say the orch runs in the prime and in any beam.

**ImpactSummary:** The orch pre-run admits beams and start gains two beam refusals; refuseNonPrime's callers are Command and its test, both on this card.
