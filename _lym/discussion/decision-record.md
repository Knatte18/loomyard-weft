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

### Added after Discussion (parent, 2026-11-23): F4: scope is connect and policy refusal

Decision: This task covers the RC button's state at connect and on a policy refusal only. A later disconnect or turn-off of remote control is out of scope.

Rationale: Issue 189 is the policy refusal; disconnect handling is a separate change.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - orch-start-waits-before-save-and-register (class design, severity MEDIUM, open in rounds 1)
  - read-failure-after-paste-requeues-command (class design, severity MEDIUM, open in rounds 1)
  - transcript-filename-literal-repeated (class consistency, severity BLOCKING, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
