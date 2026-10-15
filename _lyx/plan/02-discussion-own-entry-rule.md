# Card 2 — discussion-own-entry-rule

**Edit:**
- `contracts/stencils/loom/loom-template-discussion.md`
- `contracts/stencils/loom/loom-rubric-discussion-review.md`

**Intent:** Carry D2 and D5's Discussion-review part.

In `loom-template-discussion.md`, add to the design rules (Step 3's paragraph closing "Design the full scope now." is the natural home) a sentence stating that no decision asks the run to change this task's own board entry's body or scope, through `lyx board` or otherwise, because the run's board lock refuses such a change.
State the bound in the same place: the landing's status write, a `run_settings` write, a decision to file or change another board entry and a change to the board module's code are outside the rule.
The chair stencil includes this stencil whole and gets the rule with no copy; the advisor stencil stays unchanged.

In `loom-rubric-discussion-review.md`, add an `## Also flag` bullet naming as a finding a Decision that asks the run to change the task's own board entry's body or scope, through `lyx board` or otherwise; a Decision that writes another entry, the entry's status or its `run_settings`, or changes board-module code is not flagged by it.

Constraints on the new text, each pinned by a test in `contracts/stencils`: no bare `contracts/`, `docs/` or `internal/` `.md`/`.go` path token in the agent-facing body (`TestStencils_NoBareCrossRepoCitations`), no ask-the-operator phrase (`TestStencils_NoUnboundedAskOperator`), no edit-rule phrase such as "edit or write" (`TestStencils_NoEditRuleRestatement`), and no command the agents' settings deny (`PATTERN-no-denied-recovery`).
Add no template marker.
Write one sentence per line.
No test is planned: a stencil's static prose is exempt under `PATTERN-test-economy`.

**ImpactSummary:** Two stencil files read at call time; the Discussion chair includes the template whole, and no Go code or test asserts the added sentences.
