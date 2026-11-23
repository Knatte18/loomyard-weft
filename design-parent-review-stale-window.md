# Parent-review asks look up the orch pane in a stale tmux window, so an approved PR review cannot end

`parent-review-stale-window` · Task · bug, landing, shuttle · running · Preflight

Depends on: [`pr-review-reask-not-delivered`](design-pr-review-reask-not-delivered.md)

Imported from [issue #206](https://github.com/Knatte18/loomyard/issues/206).

Seen in hub mo, beam dop-support-3, run hydraulics-gaps, at PR-Review with pr_review_ask_parent on. The parent orch strand mo:dop-support-3:orch is live at pane %10 in window @8, but lym loom parent-review ask records each ask from the PR-Review session as no-parent, apparently looking for the orch in window @4. The asks the row itself sends at a new head are delivered: asks 3 and 7 were answered. lym loom parent-review answer then refuses a no-parent ask ('ask 9 is already no-parent; ... wait for the next [pr-review request]'), and lym loom pr-continue refuses with 'no satisfied answer from your parent stands at <head>'. Once the parent is satisfied at an unchanged head, the review therefore cannot end. Workaround: turn pr_review_ask_parent off for the run with lym config loom --layer local --set pr_review_ask_parent=false in the task worktree, then have the PR-Review agent run pr-continue. Expected: the ask resolves the orch's current pane, and a parent can answer, or record a satisfied verdict for, the head that a no-parent ask named. Related: #200.