# Add autonomy mode: the parent orch settles halts, operator sees severe

## What changes

A new `beam.yaml` key `autonomy` (boolean, default `false`, `CategoryNone`) puts a worktree's runs under their parent orch.
Under it the orch settles every halt and run-level call itself.
The operator is reached for two things only: a severe halt, and a merge-ready beam.

Autonomy is looked up from the worktree upward through the worktree chain to the hub's default layer, via the per-task lookup in `dobbyengine`.
A task sets its own value through `run_settings` (`{"beam":{"autonomy":true}}` or `false`), and a set value wins in both directions over its parent.
Finalize's config-change notice leaves `autonomy` out, so a landing never asks an orch to re-apply a child's value on its parent.

Concrete effects:

- Every halt notice carries `autonomy: <true|false>`, the run's value read when the notice is built.
  The orch handles a halt under that value, not its own worktree's.
  A stack that fails to load reads `false` and logs a warning.
- Autonomy forces `pr_review_ask_parent`, `judge_ask_parent` and `raddle_auto_approve` to `true`,
  and `judge_interactive` and `discussion_interactive` to `false`.
  No layer file is rewritten, so turning autonomy off restores each key's own value at the next load.
- Drift-Verify halts when a judge ask ends without the parent's answer (timeout, not-sent, no-parent).
  `lym loom resume` clears the ended ask and sends a new one.
  Without autonomy the rulings stand, as before.
- PR-Review records the run's autonomy once per review in the pull request record (`internal/pullrequest`).
  Under a recorded `true` the review session is non-interactive and ends only when `lym loom pr-continue` sees a satisfied parent answer at the gated head.
  The new `lym loom parent-review ask` sends each re-ask after fixes, and `lym loom parent-review answer` records the parent's answer.
  A new loom key `pr_review_parent_timeout_min` (default 60) sets the ask's deadline.
  A commit after a passing `pr-continue` leaves the review open until a new ask at the new head is answered satisfied.
  A closed pull request halts PR-Review, and halts Finalize when closed after the review ended; neither reopens it.
  `lym loom goto` onto PR-Review clears the record's autonomy and session so the re-entry starts fresh.
- Beam-Await posts one `[beam drained] <slug>` line to an autonomous beam's orch once the beam has commits beyond its parent, no open member, no child pair and no ready record.
  A drain marker file under the beam's `.lym/beam/` directory prevents repeats.
- The Slack reply hook forwards only replies opening `[operator]` while the orch's autonomy is `true`.
- Every orch pointer (start, resume, reload, and the pending resume after a handoff) carries a line naming the orch's autonomy.
- New `contracts/orchrole/role-autonomy.md` holds the orch's autonomy handling:
  the six severe-halt classes, the `[operator] severe halt <slug>:` opening, the merge-ready steps, and the direct Slack posts.
  `orch-role.md`, `role-slack.md` and the `lym beam pr` and `lym beam describe` help name it as the exception to their operator-word rules.
  `role-slack.md` also gains the `[operator]` marker rule, which holds whatever the autonomy.
- Two glossary lines: **autonomy** and **severe halt**.

No issue is closed by this change.

## Design decisions

- `autonomy` is not a Guard key, by the operator's call.
  A run's agents could edit their own worktree's file; their role instructions forbid it.
- The severe-halt list is closed at six classes: owner halt or takeover, scope change, `lym batten cancel`, a step only the operator can take outside lym, a repeating stop, and a halt with no settling action.
  Class 5 counts earlier halts at the row from the run's status history, each Stuck entry with no `routed_to` counting as one plus its `repeats`.
- Autonomy does not turn member pull requests off, does not merge (Finalize alone merges), and leaves draft release, the operator's own task filing and note promotion with the operator.
- `code_review_ask_parent` and `discussion_review_ask_parent` are not forced: they add a veto but remove no operator wait.
- On `[beam drained]` the orch runs the landing steps up to the pull request (merge-parent only on a conflict refusal, describe, `lym beam pr open`) and notifies the operator.
  `lym beam ready` still waits for the operator's word.
- A pull request record that holds autonomy gains a field older readers have not seen; the field is additive.

## To check by hand

- Turning autonomy on in the Models hub on OSL-1046 or in loomyard is a config write the operator orders after landing; this change does not do it.
- Finding `goto-stop-reads-unwired-reed-runner` (scope, blocking, plan stage): check that the goto stop path does not read a reed runner that is unwired.
- Finding `clear-review-session-keeps-parent-review-asked` (design, medium): check that `ClearReviewSession` in `internal/pullrequest` does not leave the PR-Review ask state set after `goto`.
- Finding `dobby-run-notices-unstamped-autonomy` (scope, medium): check that every dobby run halt notice carries `autonomy:`.
- Finding `goto-hook-wiring-untested` (scope, medium): check that the goto spec's hook that clears the PR-Review record is wired and covered by a test.
- Finding `halt-notice-call-sites-untested` (scope, medium): check that each halt-notice call site is covered by a test that pins its `autonomy:` stamp.
- No regression check ran for this task, and no integration suite section is in the run record.

## Merge danger

**Door:** two-way
**Blast radius:** orch
