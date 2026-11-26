## Goal

```markdown
Imported from [issue #264](https://github.com/Knatte18/loomyard/issues/264).

## What happened

The first gate run of a batch failed only on a `lym board` process that a `TestLoomPreBootstrap` fixture left running in `internal/loomcli`.
The rerun passed with no change.

## Why it is abnormal

A test leaves a process behind, and the gate fails on it intermittently, unrelated to the change under test.

## What to do

Have the fixture stop, or wait for, every `lym board` process it starts before the test returns.
The relevant fixture is around `internal/loomcli/smoke_prebootstrap_integration_test.go`.

Provenance: task `config-keys-nested`, note `10-loom-key-messages.md`.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-26): No test seam: the bug is in test-only fixture code

Decision: The red-first gate cannot judge this fix: the defect and the fix both live in internal/loomcli _test.go files (waitForFixtureProcessesToExit in smoke_helpers_test.go, newBadReedUpFixture), and the production code is unchanged, so no test can fail on the base's production code.

Rationale: The leaked detached 'lym board sync' came from test fixture code alone; the base's production code has no bug for a test to catch.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - tier2-fixture-callers-leave-board-sync-running (class scope, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
