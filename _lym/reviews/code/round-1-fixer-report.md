# Fixer report — code round 1

## Changes

- **F1 (NIT) — fixed as proposed**, commit `d62ee2e3`.
  `internal/hubforge/hub_test.go`: `TestInitRepos_WriteFixtureConfigKeys` gained an `absent` field.
  The Bare row sets it to `user.name` and `user.email`.
  The test asserts that `git config --local --get <key>` exits 1 for each.
  `gitkit.Git` fatals on a nonzero exit and `gitExit` is unexported, so the check calls `exec.Command` directly.
  `lym gate test --tier 1 ./internal/hubforge` passed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
