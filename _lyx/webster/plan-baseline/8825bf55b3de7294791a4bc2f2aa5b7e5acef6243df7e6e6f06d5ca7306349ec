# Card 6 — Orch briefs carry the gate rule to non-fork agents

**Edit:**
- `contracts/stencils/orch/orch-template-role.md`
- `contracts/stencils/claims_test.go`

**Intent:**
Decision record D6: an orch fork inherits the gate rule through the shared system prompt, but a fresh non-fork agent does not see the orch's appended system prompt, so the orch carries the rule into that agent's brief itself.

In `## Fixing a run`, after "You run no test suite yourself, neither `lyx gate test` nor a reproduction run: a test run or a reproduction goes to a fork or an agent, since it takes too long in your session.", add:
"A brief for a non-fork agent that runs `lyx gate test` carries a copy of the gate rule from your own system prompt, its timeout value included, since only a fork inherits that prompt."
The stencil names where the orch copies the rule from and never spells its content, so the runtime copy is the one sanctioned restatement outside the steer.

`claims_test.go`, in the `orch-template-role.md` entry: an `inSection("## Fixing a run", …)` claim pins "carries a copy of the gate rule from your own system prompt, its timeout value included", with a `why` saying a non-fork agent does not inherit the orch's system prompt.

**ImpactSummary:** the orch role stencil gains one sentence and the claims test one row.

**Commit:** 6: orch briefs carry the gate rule to non-fork agents
