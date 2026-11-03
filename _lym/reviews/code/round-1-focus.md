---
round: 1
focus:
  - "internal/driftengine/baseline.go `drive` and `settleConfiguredScope`: settle whether a configured scope that is recorded before any capture leaves every later path consistent, in particular a resume after a parent replaced the scope, a recorded scope that differs from `cfg.Scope`, and a pass where `scope.json` exists but `baseline.md` does not."
  - "internal/driftengine/baseline.go `drive`: settle whether the new `d.Config()` call and its Stuck outcome are reached with drift switched off, and whether that path still spawns, starts and commits nothing as `Call`'s doc comment says."
  - "internal/driftengine/baseline.go `settleConfiguredScope`: settle whether `UpdateScope` with the `errNoRecord` sentinel really leaves the record untouched when one is found, and whether any other error from it is wrongly swallowed or surfaced."
  - "internal/driftengine/config.go `LoadConfig` and `checkScope`: settle whether the `scope` and `skip` keys read through the hub-wide layer stack from a board entry's `run_settings` as `{\"drift\": {\"scope\": ..., \"skip\": [...]}}`, and whether the `CategoryNone` registration is the right category for them against `configengine`'s rules."
  - "internal/driftengine/config.go `checkScope`: settle whether the three issue shapes (full, partial with skip, off) all load, and whether the refusals match the issue's wording; also whether `Skip` holding a non-nil empty slice or duplicates can reach `ScopeRecord` in a form `ReadScope` or the compare later rejects."
  - "Scope against the issue: settle whether the diff changes anything the issue does not ask for (the `contracts/orchrole/orch-role.md`, `docs/overview.md`, `driftcli/cli.go` and `doc.go` edits) and whether the docs the repository's constraints name for this kind of change landed in the same diff."
  - "Test economy in internal/driftengine/baseline_test.go and config_test.go: settle which new tests cover the no-agent path, the kept-recorded-scope path and the refusals, whether any duplicates another, and whether any new behavior (the missing-`baseline.md` write, the `configuredBaselineReport` text) has no coverage."
  - "The commit message and any claim in `_lym/landing/summary.md`: settle whether 'a scope already recorded is kept' and 'writes baseline.md itself' are borne out by the diff."
---

Seed focus for round 1. The answer key is the decision record's `## Goal` alone, with no `Added after Discussion` entries.
The diff is one commit over `e3b062ab`.
