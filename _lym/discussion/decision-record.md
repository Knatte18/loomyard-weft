## Goal

```markdown
## What was seen (operator, 2026-11-22)

Bringing up the scribe hub's orch with `lym beam start` turned on Claude Code's remote control for it, because `internal/orchengine/template.yaml` ships `remote_control: true`.
The operator decides when a session gets remote control; lym must never turn it on by itself.

## Fix

- `remote_control` defaults to `false` in the orch template, so no hub gets remote control on any launch or resume unless its config turns it on.
- Every hub that has no explicit value takes the new default; ship the reinterpretation `PATTERN-config-layers` asks for a meaning change.
- The RC button of a pane stays the operator's way to turn it on for one agent.
- Check that no other launch path, such as a beam orch or a respawn, turns remote control on by its own default.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-22): F1: no transition window for remote control

Decision: The window is refused. In this run, reed's revival of an orch strand (Resume in internal/reedengine/lifecycle.go and the absent-pane relaunch in strand.go) drops the remote-control argument from the stored resume line while orch.yaml's remote_control is off, so no launch, resume or revival turns remote control on unless the hub's config asks.

Rationale: The operator: lym must never turn remote control on by itself, not on any path; the entry says no launch or resume.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - stored-resume-line-keeps-remote-control (class decision, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
