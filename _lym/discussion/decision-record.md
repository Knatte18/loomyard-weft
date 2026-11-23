## Goal

```markdown
Imported from [issue #189](https://github.com/Knatte18/loomyard/issues/189).

## Seen

In the DrillOpPlan hub (lym build 30ed1ee58802), the RC button of the orch pane shows green, as if Remote Control were on.
Remote Control is off: the organization's policy disables it, and `/remote-control` in the same session answers "Remote Control is disabled by your organization's policy."
The hub's `orch.yaml` sets no `remote_control`, and after this build its default is off, as the reconcile notice said.

Clicking the button makes reed answer that Remote Control is already on, which it is not (operator, 2026-11-23).

## Expected

The RC button shows Remote Control's real state for the session.
Where the organization's policy disables Remote Control, the button shows it off or unavailable, and a click does not claim it is on.

## Workaround

None needed: the operator ignores the button.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
