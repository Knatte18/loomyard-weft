# Card 13 — decision-by-arbiter

**Edit:**
- `internal/loomcli#decisionDeps`
- `internal/loomcli#decisionVerb`
- `internal/loomcli#decisionState.preRun`
- `internal/loomcli#loomCLI.decisionCmd`
- `internal/loomcli#TestDecisionVerb_AppendsAndCommits`
- `internal/loomcli#TestDecisionVerb_Refusals`
- `internal/loomcli/circling.go`
- `contracts/specs/refusal-spec.md`

**Create:**
- `internal/loomcli#decisionByArbiter`

**Uses:**
- `internal/loomengine#IsArbiterStrand`

**Intent:**
Let the run's own arbiter record a design call with `lyx loom decision add --by arbiter` (D12).

`decisionDeps` gains the field `arbiterStrand func(name string) bool`, true for the target run's own arbiter strand.
`decisionState.preRun` fills it from `hubgeom.ReedGeometry(target)`: a name `agentname.Parse` accepts whose shortname and slug are the geometry's and that `loomengine.IsArbiterStrand` accepts.
In `decisionVerb`:

- `--by` admits `arbiter` beside `parent` and `operator`; the unknown-value refusal keeps its wording, since `arbiter` is not a label other callers may pass;
- `--by arbiter` passes the caller check only when the exported strand name is non-empty and `arbiterStrand` accepts it; from any other caller, the operator's unnamed shell included, it is refused with a way forward: the label is the run's arbiter's, so pass `--by parent` from the run's parent or `--by operator` from the operator's shell;
- an arbiter strand passing `--by parent` or `--by operator` is refused as a task session is today;
- `--by parent` and `--by operator` behave as before.

Bound: the label stays unverified beyond the strand-name check, as `--by parent` is; the entry stays append-only, dated and committed.
`decisionCmd`'s help names the `arbiter` label and who may pass it.

Add a `TestDecisionVerb_AppendsAndCommits` row for the arbiter strand with `--by arbiter`, the heading carrying the `arbiter` label, and `TestDecisionVerb_Refusals` rows for `--by arbiter` from the operator's shell, from the parent, and from another task strand, and for the arbiter strand passing `--by parent`.

Update `circling.go`'s file doc comment (D3): the verbs' unchecked callers are bounded by which stencils name them, now the Bouncer escalation brief and the arbiter stencil, and by D2's Bouncer-escalation eligibility and the verbs' own preconditions.
Add a loom-section row to `contracts/specs/refusal-spec.md`: `decision add: --by arbiter from another caller`, a correctness halt quoting its way forward.

**ImpactSummary:** the decision verb admits one more caller under one label; `--by parent` and `--by operator` are unchanged.
