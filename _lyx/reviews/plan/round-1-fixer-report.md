# Plan-Review round 1 — fixer report

All four findings fixed; `lyx loom validate-plan` returns `ok: true` over the edited plan.
`approved:` in `00-overview.md` is untouched.

## Changed

- **F1 (MEDIUM):** `01-refuse-before-archive.md`: `TestRemove_WeftDirtyRefusalPushesNoTagAndResumeArchivesOnce` now also asserts, after the refused call, that the error does not contain `lyx fabric reconcile`, that `res.Mutated().Len()` is 0, and that the portal link, launcher dir, both worktrees, the warp branch and the weft branch exist, covering all of acceptance criterion 1.
- **F2 (LOW):** `00-overview.md`: the verify section's integration line is now `go test -tags integration ./...`, the full suite the decision record's open risk calls for.
- **F3 (LOW):** `01-refuse-before-archive.md`: the ImpactSummary now names both production callers (`fabriccli`'s `fabric remove`, `battencli`'s teardown) and the change they observe: a refused call leaves the pair and weft origin untouched.
- **F4 (NIT):** `01-refuse-before-archive.md`: `TestRemove_ReusesSameTipArchiveTag` now plants a lightweight tag (`git tag <name> <tip>`), with the reason: `archiveWeftTip`'s reuse check compares the ref's objectname to the tip.

## Deferred

None.
