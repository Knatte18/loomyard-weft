```yaml
lens: fix-consistency
outcome: reported
```

No findings.

I checked three points, and the change set agrees with its surroundings on all of them:
- `boardWith` is a local closure used only in `TestReedGeometry_ButtonContext`. All four call sites now pass the new slug argument.
- The slug `child-task` in the rows matches the worktree name that `hubWithOrigin` sets (`parent_test.go:122`). The `BoardBeamOfAnotherSlugReadsTask` row's `other-beam` slug is therefore a real mismatch against `pairIsBeam`.
- The test's doc comment and the `pairIsBeam` doc comment still describe the behaviour the rows pin.

Tool calls:
1. `grep` for `boardWith(` and `child-task` in `internal/hubgeom/*_test.go`. I needed it because my context lacked the slug the `hubWithOrigin` helper uses.
