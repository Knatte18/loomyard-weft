# Card 12 — orch-stencil-alignment

**Edit:**
- `contracts/stencils/orch/orch-template-role.md`
- `contracts/stencils#wordingClaims`
- `CLAUDE.md`

**Intent:**
Make D8's orch role changes and D5's production paragraph change, so the orch knows parent review and PR-Gate reach it only as escalations, and records operator decisions where the alignment agent reads them.

`orch-template-role.md`:
- `## Parent-review`: a parent-review request reaches the orch only when the alignment agent escalated or gave no verdict. The brief names the alignment report, which the fork reads first, or why no verdict came. The existing fork, repeat-notice and no-message rules stay.
- `## PR-Gate`: an awaiting `PR-Gate` reaches the orch only when the agent escalated, gave no verdict, or approved under `alignment_pr_approve: false`. The awaiting notice names the alignment report the fork reads first. An agent-approved run lands with no orch turn. The orch amends the board entries that a queued alignment notice names as superseded, in place of "after approving, amend the board entries the run superseded" for its own approvals.
- `## Escalations from a child`: an escalation naming a peer that has its own open escalation is handled together with that peer's.
- `## The board` gains the decision-recording rule. An operator decision made in chat is recorded in the same turn, split by holder:
  - A decision on an entry no run holds is written into that entry's body through `lyx board`.
  - A decision on a task a run holds is recorded with `lyx loom decision add <slug> --by parent`.
  - When that verb refuses because the run has no decision record (batten's pre-loom rows such as `running · Worktree-Create`, `Preflight` and `Loom-Preflight`, a halt before Discussion-Write, or any darn run), the decision becomes a board note of its own through `lyx board upsert` with kind note, naming the held task's slug.
  - Only when the verb refuses because Discussion-Write is running and owns the record does the decision go by message to the Discussion-Write session.
  - The board's run lock is unchanged.
- `## After landing`: while the hub's landing `deploy` key is empty, the orch runs `update-plugins.sh` on the landing notice, or at its next wake after any landing, instead of at its next wake after an approval, since an agent approval lands with no orch turn.

`wordingClaims`' `orch-template-role.md` row gains claims:
- `alignment agent escalated` in `## Parent-review`.
- `lands with no orch turn` and `superseded` in `## PR-Gate`.
- `lyx loom decision add <slug> --by parent` and `kind note` in `## The board`.
- `after any landing` in `## After landing`.

The claim forbidding `alignment-agent` (`contracts/stencils/claims_test.go`, in the same row) keeps its `mustNot`, and its why is rewritten to say that the stencil names the alignment agent in prose and never by its board-note slug.

Each claim carries its reason, and the existing claims keep passing.
The `## PR-Gate` claim on `judges the run against its own board goal and the operator's recorded decisions` stays, since the fork still runs on an escalation.

`CLAUDE.md`'s production paragraph adds the second standing route next to the orchestrator's exception.
The alignment agent's approve at PR-Gate lands a task with no orch turn, and batten's Land row then runs the hub's `deploy`.
`alignment_pr_approve: false` in the hub's `loom.yaml` withdraws that landing.
The paragraph keeps one sentence per line.

**ImpactSummary:** Stencil and doc text only, with the claims test table extended to pin the new orch rules; no Go behavior changes.
