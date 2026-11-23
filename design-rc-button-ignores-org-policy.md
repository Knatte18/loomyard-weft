# RC button shows green while Remote Control is disabled by organization policy

`rc-button-ignores-org-policy` · Note · bug, reed, claude

Imported from [issue #189](https://github.com/Knatte18/loomyard/issues/189).

## Seen

In the DrillOpPlan hub (lym build 30ed1ee58802), the RC button of the orch pane shows green, as if Remote Control were on.
Remote Control is off: the organization's policy disables it, and `/remote-control` in the same session answers "Remote Control is disabled by your organization's policy."
The hub's `orch.yaml` sets no `remote_control`, and after this build its default is off, as the reconcile notice said.

## Expected

The RC button shows Remote Control's real state for the session.
Where the organization's policy disables Remote Control, the button shows it off or unavailable, and a click does not claim to turn it on.

## Workaround

None needed: the operator ignores the button.
