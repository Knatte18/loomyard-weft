# Card 10 — pr-review-stencil

**Create:**
- `contracts/stencils/landing/landing-template-pr-review.md`
- `contracts/stencils#LandingTemplatePRReview`

**Edit:**
- `contracts/stencils#entries`
- `contracts/stencils#roleOpeningStencils`

**Uses:**
- `contracts/stencils#LandingTemplateFixReview`
- `contracts/stencils/landing/landing-template-fix-review.md`
- `contracts/stencils#TestRegistry_MatchesOnDiskTree`

**Intent:**
D7 and D13: the PR-Review session's first prompt, one stencil for the loom, darn and pawl recipes, read at call time through `internal/stencilstore`.
The new `landing-template-pr-review.md` opens with a leading comment in the shape of `landing-template-fix-review.md`'s, saying who fills it (landingshed's PR-Review row) and which markers are required; then `{{.parent_directive}}` and `{{.edit_directive}}`.
Its markers, each a top-level substitution with no conditional: `slug`, `pr_url`, `task_worktree`, `task_branch`, `parent_branch`, `description_path` and `run_material`, a list of the paths the run has among the decision record, the plan directory and the board entry snapshot, which the row renders, one per line.
It tells the session, in paths only: it is the agent the operator discusses pull request `{{.pr_url}}` with; it reads the change (`git diff {{.parent_branch}}...HEAD` in `{{.task_worktree}}`), the change description and the listed material itself; it may commit to the task branch `{{.task_branch}}` and lym gates and pushes its commits, and mails it here when a commit fails the gate; it never pushes, never approves, merges or closes the pull request and never calls GitHub, which its settings also deny; the run ends on the pull request's state, not on the session, so it waits for the operator's next message after each answer.
It names the operator's ending: when the operator says they are satisfied and asks it to continue, it commits or discards its work so the worktree is clean, then runs `lym loom pr-continue`, which closes the pull request and lands the task through the ordinary landing with its gate.
It never tells the session to ask the operator anything outside its own pane conversation, so `TestStencils_NoUnboundedAskOperator` holds.
`stencils.go` embeds it as `LandingTemplatePRReview` beside `LandingTemplateFixReview`, lists it in `entries` as `landing-template-pr-review`, and maps the role `pr-reviewer` to it in `roleOpeningStencils`, so the parent-directive and edit-directive scans cover it.
`TestRegistry_MatchesOnDiskTree` covers the new file through the registry.

**ImpactSummary:** One new embedded stencil registered in the two tables; the existing stencils and their scans are unchanged.

**Commit:** 10: stencils: the PR-Review session's opening stencil
