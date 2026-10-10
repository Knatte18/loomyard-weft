# Card 7 — decision-add-refuses-task-session

**Edit:**
- `internal/loomcli#decisionDeps`
- `internal/loomcli#decisionVerb`
- `internal/loomcli#decisionState.preRun`
- `internal/loomcli#loomCLI.decisionCmd`
- `internal/loomcli/decision_test.go`
- `contracts/stencils/claims_test.go`

**Prosa:**
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`

**Uses:**
- `internal/hubgeom#ResolveParent`
- `internal/agentname#StrandNameEnv`

**Intent:** `lyx loom decision add` refuses both `--by` labels from a named session other than the run's recorded parent (D6).
`decisionDeps` gains `strandName string`, the full name reed exported to the calling session, and `parent func() (hubgeom.Parent, error)`, the run's recorded parent; `preRun` fills them from the strand-name environment key through `os.Getenv` and from `hubgeom.ResolveParent` over the resolved target, deferred to call time so a test needs no hub.
`decisionVerb` checks after the flag checks and before the status read: an empty `strandName` passes; otherwise it resolves the parent; a resolution error refuses with `way forward: the run's parent could not be resolved (<err>); the operator's own unnamed shell may still run the verb`; a name equal to the resolved parent's name passes, a slug-bearing parent of a pair created from a task worktree included; any other name refuses, naming the caller, with `way forward: a design call after the Discussion is the parent's to record; ask <parent name> to run the verb` when a parent is recorded and `... is the operator's to record; the operator's own unnamed shell may run it` when none is; the message never spells a hub's shortname as a literal of its own, only the resolved name.
A refusal appends nothing and commits nothing.
The file header comment, the group's `Long` and `add`'s `Long` drop the sentence that the verb checks no caller identity and say instead that it refuses a task session's label.
`TestDecisionVerb_Refusals` gains rows: a named caller differing from the recorded parent; a named caller in a run with no recorded parent; a named caller whose parent resolution errors, each asserting the way forward, no append and no commit.
`TestDecisionVerb_AppendsAndCommits` gains rows: an empty name appends; the recorded parent's own name appends; a slug-bearing recorded parent's own name appends.
The refusal-spec's loom section gains `decision: task session` (trigger: `lyx loom decision add` runs under a non-empty strand name other than the run's recorded parent name, or one whose parent cannot be resolved; correctness halt; the way forward clauses above), beside the other `decision:` rows.
`docs/overview.md`'s `decision add` line replaces "checks no caller identity" with the refusal.
The plan stencil's scope-addition paragraph is replaced by: a scope addition that arrives after the Discussion is a design call the parent records; ask your parent to record it and plan on it once the record carries it, never write it into the plan as an operator addition, and with no parent recorded, stop and report the addition instead of planning on it; it names no hub shortname and no longer names `lyx loom decision add`.
The claims table for `loom-template-plan.md` replaces its `decision add` claim with a `wantNone` for `lyx loom decision add` and a `wantAll` for `ask your parent to record it`, `Never write such an addition into the plan as an operator addition` and `stop and report the addition`.

**ImpactSummary:** One verb gains a caller check with two injected inputs; its only production constructor is `preRun`, and the stencil change reaches the Plan producer through `PlanSpec` unchanged.
