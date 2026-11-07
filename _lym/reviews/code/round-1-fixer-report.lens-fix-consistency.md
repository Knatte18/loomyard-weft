```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set (the extended `TestEnsure_HubWideWalkRendersAndCommitsTheHeddleOverview` in `internal/hubreconcile/ensure_integration_test.go`) against its surroundings, through the fix-consistency lens.

Fork finding list (lens fix-consistency):

1. LOW — `commitRetiredLandingKey` in the new test duplicates `seedRetiredHubWideKey`.
   - Changed text: the `commitRetiredLandingKey` closure at the top of the rewritten test.
   - Evidence: `seedRetiredHubWideKey(t, h)` at `ensure_integration_test.go:419` already commits `ConfigFileRel("landing")` with `"publish_verify: x\n"` and the message `"fixture: retired landing key"`. The closure repeats that literal and path, so a change to the retired key would have to be made in two places.
   - The helper takes a `*hubforge.Hub`, while `seed` receives only `boardDir`, so reuse would need `seed` to take `h` or the helper to take `boardDir`. This is a stale twin, not a wrong behaviour.

Everything else agrees with its surroundings:
- Rows 1 and 2 keep the original assertions through `failedWriteFirst` and `wantOverviewCommitted`.
- `landingRel` uses `filepath.ToSlash`, which matches the other tests' commit-list checks.
- The field comments follow the table-field style used elsewhere in the file.
- The `walk.go` and `doc.go` contracts (a failed render is logged and the walk goes on; an identical render adds nothing to the commit) match what rows 3 and 4 assert.
- The review's F1 and F2 fix text maps one-to-one onto the new rows.
- The gate run you reported passed with the new rows executed.

Tool calls made: none, because the context already held everything this lens needed.
