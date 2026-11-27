---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
---

# Fixer report, code round 1

## Changes

- **F1** (`ef9ddf931`): the row "a dirty worktree diffs from the parent merge base although the record holds a whole-green ancestor"
  in `internal/gatecli/scopes_integration_test.go` now commits a change to `other/other.go` before its `--all` run,
  so the whole-green entry is for a commit other than the merge base.
- **F2** (no commit): `_lym/landing/summary.md` now says the plan sits under `_lym/plan`,
  which the pair's code repository excludes.
- **F3** (`1cf73dbd9`): the `scenarioHub` doc comment in `internal/gatecli/test_integration_test.go`
  now says every row that runs in a hub builds its own.
- **Self-check follow-ups** (`b48c059f2`, and `_lym/landing/summary.md`): the `scopesHub.plan` field comment
  carries F2's wording, and the summary's bullet on rows that set up their own state names F1's row.

`lym gate test --tier 1 ./internal/gatecli -- -run '^TestGateTest_(Scopes|Scenario)$' -count=1`
passed after the last change: 142 discovered, 142 passed, 0 skipped.
F1's row was not run against a gate with the defect injected.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW, the `scopesHub.plan` comment still said "never-tracked": fixed as proposed.
- fix-consistency, LOW, the summary did not name F1's new setup step: fixed.
- fix-proposal, LOW, the `scopesHub.plan` comment (same as the first fix-consistency finding): fixed.
- fix-affected-set, LOW, the `scopesHub.plan` comment (same finding again): fixed as proposed.

## Disputed

## Deferred

## Rulings
