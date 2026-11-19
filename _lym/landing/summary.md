# Run agents ask their parent orch; the parent can subscribe to rows

A run's autonomous agents now put their questions to the run's parent orch by creel instead of to the operator in their pane.
The parent can also ask lym to mail it when the run reaches a named row, so it can brief that row's agent as it starts.
Both come from the comment on #153, where the parent saw an agent's question only after the operator relayed it,
and had to poll `lym loom status --watch` to catch a row's start.

## Questions go to the parent

- `contracts/orchrole/orch-role.md` gains a "Questions from a run's agents" section.
  The parent answers a run-level call itself and puts a design question to the operator first.
  An agent that asked in its pane anyway reaches the parent as a hold notice.
  An interactive session the operator chose to sit in, such as Discussion, PR-Review, a sampler or a `judge_interactive` judge, still asks in its pane.
- `parent-directive-operator-ban.md` now tells the agent to put every question to its parent rather than only banning the pane question.
- `bouncer-fragment-decision.md` and `burler-step-fix.md` no longer send a finding or a deferred decision to the operator.
  The parent settles a run-level call and asks the operator only for a design question.

## Row subscriptions

- New `notify_parent_on` key in `loom.yaml`, open to a task's `run_settings`: a list of the recipe's row names, empty by default.
  `internal/loomengine` registers the key in `ConfigSpec` and `template.yaml`.
- `newShed` in `internal/loomcli` wraps each named row's producer (`rownotice.go`).
  Before the row's call it posts a `[row reached]` line to the parent as `<shortname>:LOOM`.
- A name that is no row of the run's recipe refuses the run's steps and names the recipe's rows.
- A failed status read, record read or write, or post only logs a warning; a notice never stops the run.

### Design decisions

- Rows only, no state subscriptions.
  The need is briefing a row's agent as it starts.
  Halt and other states already reach the parent through batten and dobby notices, so a state subscription would duplicate them.
- The key is chosen over a `lym loom notify` verb, so a task can set it through `run_settings` with no extra command.
- "Once per visit" is the count of verdicts the run's history holds when the row is called, kept in a new ephemeral `row-notice.json` under the run's scratch dir (`shedrun.RowNoticeRecord`).
  A call after a yield or a pause records no verdict, so it sends no second notice.
  A re-call after an escalation's answer or a resumed halt spawns a fresh agent and does send one.
  A row the entry gate skips is never called and sends none.
- The visit is recorded before the post, so a failed post is never retried.

## To check by hand

- `card-fixer-notifier-never-fires` (design, medium): a review finding that a fixer's notifier may never fire.
  Confirm that a `notify_parent_on` row naming a review fixer sends its notice in a live run.
- `newshed-notify-wiring-untested` (design, medium): a review finding that `newShed`'s notify wiring in `internal/loomcli/arm.go` lacked a test.
  `rownotice_test.go` now covers the wrapper and its contracts; confirm it exercises the `newShed` wiring through `c.env.NotifyParent`.
- No end-to-end run with `notify_parent_on` set was done: send a real run through a named row and confirm the parent receives the `[row reached]` mail.
