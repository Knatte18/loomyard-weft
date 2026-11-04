# A PR-Review waiting on the operator shows nowhere

`pr-review-wait-silent` · Note · bug, landing, loom, batten · draft

## Seen

On the `cs` hub, task `cached-friction-factor` (Drift trial 2, 2026-11-03) reached PR-Review at 18:53:02Z and opened PR #4.
Its PR-Review session then waited for the operator for 21 minutes, until `cs:orch` told it at 19:14:26Z that the operator had turned PR review off (creel 35); the row ended done at 19:14:35Z.

Through those 21 minutes nothing surfaced that the run waited for a human:

- `lym loom status --watch` in the task worktree printed `now PR-Review | last Describe → done` once and no `waiting` line after it.
- `lym batten status cached-friction-factor --watch` showed `running | now Run-Shed` throughout.
- The `cs` creel journal holds no post between Drift-Recheck and creel 35: no batten notice, no notice to `cs:orch` and none to the loomyard hub.

A watcher reading the documented status surfaces could not tell a PR waiting on the operator from a row at work.
The first trial (`docs/research/drift-csharp-trial-2026-11-03.md`) saw the same: "PR-Review waited 32 minutes for the operator".

## Expected

- While PR-Review waits for the operator, `lym loom status` shows a `waiting` line naming the pull request and that it waits for a human review, as other rows show their waits.
- The hub's orch gets one notice when a PR-Review starts waiting on the operator, naming the PR, so the wait reaches someone who can act.
