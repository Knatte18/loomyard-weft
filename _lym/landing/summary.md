# Run agents ask their parent; the parent can hear of a named row's start

Run agents now put any question they would ask the operator to the run's parent orch, and the parent answers itself or asks the operator.
A parent can also name recipe rows in `notify_parent_on`, and lym mails it as the run reaches each one, so it can brief that row's agent without polling `lym loom status --watch`.

## What changes

- **The parent directive.**
  The operator-ban line every autonomous role's prompt carries (`contracts/stencils/parent/parent-directive-operator-ban.md`) now says what to do as well as what not to do.
  The agent puts every question it would ask the operator to its parent with `lym creel post --parent --expect-reply`.
  The parent answers itself or asks the operator.
  A question in the agent's own pane is still never the way forward.
  The drift judge, Drift-Expect, review fixers and every other seat that renders the directive get the line.
- **Gate card fixer.**
  `internal/gatecli`'s strand rig had no hold notifier, so a cold card fixer that asked in its pane held silently until its timeout.
  It now mails each held turn end to the orch above the worktree through `orchcli.NotifyAsker`, signed `<shortname>:GATE`, as the loom, burler and webster runners already do.
- **Stencils the parent reads.**
  `bouncer-fragment-decision` now tells the parent to settle run-level findings itself and to take only design questions to the operator.
  Before, it put every remaining finding to the operator.
  `burler-step-fix` now defers a tradeoff decision to the run's parent, not the operator.
- **`notify_parent_on`**, a new `loom.yaml` key, an empty list by default and open to `run_settings`.
  It lists recipe row names, such as `["Drift-Verify"]`.
  `newShed` (`internal/loomcli/arm.go`) wraps each named row's producer (`internal/loomcli/rownotice.go`).
  Before the row's call, the wrapper mails the parent `loom <slug>: [row reached] the run reached <row> and calls it now; …`.
  It mails once per visit.
  A visit is the count of verdicts in the run's history, folded repeats included, and is kept in the new scratch record `shedrun.RowNoticeRecord`.
  A call after a yield or a pause therefore mails nothing again, but a bounce back into the row mails again.
  A row the entry gate skips is never called, so it mails nothing.
  A name that is no row of the run's recipe refuses the shed, and the refusal lists the recipe's rows.
  A failed read, write or post only warns.
- `contracts/orchrole/orch-role.md` gains the sections "Questions from a run's agents" and "Briefing a row's agent".
  `docs/overview.md` and the package docs of `loomcli`, `gatecli` and `shedrun` describe the new behavior.

## Design decisions

The decision record holds no Discussion decisions, so these calls were made here.

- **A `run_settings` key, not a verb.**
  The board entry left the choice between `lym loom notify <slug> --on …` and a key open.
  The key reuses the existing config layers and per-run `run_settings`, and needs no new subscription store or verb.
  It is a `CategoryNone` key because it only adds a mail and touches no gate.
- **Rows only, not states.**
  The key names recipe rows.
  A run's halt states already reach the parent through batten and dobby notices.
- **When the notice fires.**
  The notice fires in the wrapped producer's call, after the entry gate.
  Hooking the commit-status seam instead would have mailed for a gated row that is then skipped, such as a Drift row with drift off.
  The visit record is persisted because a run's steps can run in separate processes, so in-memory state would mail again on every yielding step.
- **Interactive sessions stay as they are.**
  Interactive Discussion, PR-Review, sampler and `judge_interactive` are sessions the operator opts into and sits in.
  The operator decision makes parent routing the default for run agents, so these opt-in modes keep asking in their pane, and the orch role says so.
- **The held-turn notice stays the fallback.**
  An agent that still asks in its pane already reaches its parent through shuttle's hold notice on every runner with a notifier.
  The gate card fixer was the one run-agent spawn path without a notifier.

## To check by hand

- No new test ran in this change. The tests are `internal/loomcli/rownotice_test.go` and the existing `never the way forward` pins in `parentdirective`, `seatengine`, `burlerengine` and `loomengine`, and the verify gate runs them.
- Set `{"loom": {"notify_parent_on": ["Drift-Verify"]}}` in a darn task's `run_settings`.
  Confirm the parent gets one `[row reached]` mail when the run reaches Drift-Verify and none at a yield or resume within the row.
  Confirm that a misspelt row refuses the run with the recipe's rows listed.
- No run has exercised the gate card fixer's hold notice end to end.
  Confirm that a held card fixer's question reaches the parent orch as a hold notice signed `<shortname>:GATE`.
