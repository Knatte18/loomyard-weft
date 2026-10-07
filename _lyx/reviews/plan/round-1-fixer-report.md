# Fixer report — plan round 1

## Fixed

- **F1 (MEDIUM):** `12-publish-push-rejected.md` now creates `internal/loomcli/wiring_publish_integration_test.go` in place of `landingdeps_integration_test.go`.
  The new test drives Publish's rejected half through `wire()`'s real seams and the production `landingDeps` closure.
  The card's regression paragraph and the `cli-drive` Shared Decision in `00-overview.md` no longer claim a missing loom-step harness.
  The resumed half stays at `landingshed`'s integration tier.
- **F2 (LOW):** `10-fabric-remote-only-commits.md`'s `ImpactSummary` no longer states a consumer count or claims the consumers sit in other packages.
- **F3 (LOW):** `11-reset-moves-remote-branch.md` now states what the remote update lets through under `SkipGit` and what bounds it.

`lyx webster validate` after the edits reports only the expected `plan-unapproved` and informational `create-new-unit` findings.
The `approved:` key is untouched.

## Deferred

Nothing.
