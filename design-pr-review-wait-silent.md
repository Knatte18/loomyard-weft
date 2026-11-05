# A PR-Review waiting on the operator shows nowhere

`pr-review-wait-silent` · Note · bug, landing, loom, batten · draft

## Operator's call (2026-11-05)

Not a problem as things stand: the operator does not want a status line or orch notice built for it now.
If it is solved, it is built only once F6 `reed-slack-relay` gives the hub a Slack link, and only as a reminder ping to the operator on Slack, such as "this has waited 30 minutes for you", repeated at an interval while the wait lasts.

## Seen

On the `cs` hub, task `cached-friction-factor` (Drift trial 2, 2026-11-03) reached PR-Review at 18:53:02Z and opened PR #4.
Its PR-Review session then waited for the operator for 21 minutes, until `cs:orch` told it at 19:14:26Z that the operator had turned PR review off (creel 35); the row ended done at 19:14:35Z.

Through those 21 minutes nothing surfaced that the run waited for a human:

- `lym loom status --watch` in the task worktree printed `now PR-Review | last Describe → done` once and no `waiting` line after it.
- `lym batten status cached-friction-factor --watch` showed `running | now Run-Shed` throughout.
- The `cs` creel journal holds no post between Drift-Recheck and creel 35: no batten notice, no notice to `cs:orch` and none to the loomyard hub.

The first trial (`docs/research/drift/csharp-trial/drift-csharp-trial-2026-11-03.md`) saw the same: "PR-Review waited 32 minutes for the operator".
