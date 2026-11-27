## Goal

```markdown
## Aim

`docs/research/speed/test-time-measure-2026-11-27/report.md` measured `internal/webstercli` at 15 to 17 s and `internal/loomcli` at 18 to 21 s of tier1 test time.

## Work

- `internal/webstercli`: its tests run one after another because they set the environment or working directory, which bars `t.Parallel`.
  Pass those values in instead, then mark the tests parallel. `TestResetCmd` (6.8 s) is the longest.
- `internal/loomcli`: share one built hub copy between the tests that only read it, and fold rows of the same shape.
- Keep every assertion; drop none.
- Measure each package's tier1 time before and after, twice each, and put both in the change description.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-27): Writer measures webstercli and loomcli tier1 time itself

Decision: The writer may run lym gate test --gate darn-write --wait --no-carry --tier 1 ./internal/<pkg> -- -count=1 in the foreground for internal/webstercli and internal/loomcli, twice each on the untouched base and twice each after its commits, and put the times with the load average at each start in the change description.

Rationale: The task exists to cut these packages' test time; only the writer's worktree holds the before and after trees, and the orch runs no tests.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - summary-loomcli-timing-claim-contradicts-table (class consistency, severity MEDIUM, open in rounds 1)
  - webstercli-serial-tests-narrowing-unrecorded (class scope, severity MEDIUM, open in rounds 1)
  - wiring-env-read-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
