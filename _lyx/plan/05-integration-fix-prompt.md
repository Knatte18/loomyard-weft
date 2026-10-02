# Card 5 — integration-fix-prompt

**Create:**
- `contracts/stencils/webster/webster-template-integration-fix.md`
- `internal/websterengine#RenderIntegrationFixPrompt`

**Edit:**
- `contracts/stencils/stencils.go`
- `internal/websterengine/render.go`
- `internal/websterengine/template_test.go`

**Intent:**
Add the integration-fix strand's prompt: a new stencil read at call time and a Go renderer for it (#332b, decision D5; Stencil Ownership Invariant).

`webster-template-integration-fix.md` is a full cold-start prompt, since the strand inherits no session context.
It follows the integration template's conventions (leading banner comment naming its renderer and markers, no conditionals, optional `{{.friction_directive}}`), and tells the strand:
- the plan's integration verify regressed after every batch landed, with the regressing test identities and their output tails (`{{.regressions}}`) and the verify command (`{{.verify}}`);
- `{{.card_hint}}`, an optional marker naming the plan card file bisect localized the regression to, as a hint only;
- fix the regressions at the current HEAD of `{{.worktree_root}}` and commit with normal dev git, in non-merge commits only;
- never write under `{{.plan_dir}}`, never commit anything under `_lyx`, and write nothing else under `_lyx` but `{{.report_path}}` and the friction note the directive names;
- never reset, rebase or amend existing commits, and never run any `lyx` command;
- leave no uncommitted change behind: Go judges only the commits, and a dirty worktree fails the attempt;
- never delete, skip or weaken a test to make it pass: a test changes only when the test itself is wrong, and the commit message says why;
- run the verify command before reporting;
- as the last action, write `{{.report_path}}` with `status: OK | FAILED` and `head_sha: <git rev-parse HEAD>` and nothing else.

In `stencils.go`, embed it as `WebsterTemplateIntegrationFix` and add its `webster-template-integration-fix` registry entry.

In `render.go`, add `IntegrationFixTemplate(stencilsDir string) ([]byte, error)` and `RenderIntegrationFixPrompt`.
The renderer refuses an empty `regressions`, `verify` or `reportPath`;
renders each regression as its identity followed by its tail in a fenced block;
renders `cardHint` as one sentence naming the card file, or nothing when it is empty (standalone mode has no bisector);
and resolves the friction directive through `friction.RoleImplementer`, swallowing a directive error as a Warn the way `RenderIntegrationPrompt` does.
Update the file doc comment's asset list and render-function list.

Tests in `template_test.go`: add the new stencil to both seeding helpers (`newTestStencilsDir`, `seedHubWebsterStencils`) and their comments;
a render test asserts the output carries each identity with its tail, the verify command, the plan directory in the never-write rule, the report path as the stated exception to the `_lyx` rule, the leave-nothing-uncommitted rule, and the no-delete/skip/weaken rule;
an empty `cardHint` renders no hint sentence and a non-empty one names the card file;
an empty `regressions` or `verify` returns an error.

**ImpactSummary:** adds one embedded stencil and one exported renderer; no existing prompt or caller changes.
