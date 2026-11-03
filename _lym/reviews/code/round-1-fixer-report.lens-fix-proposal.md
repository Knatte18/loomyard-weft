```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1–F5 as proposed, F6 by the parent's decision) through the fix-proposal lens.

Proposals checked: F1, F2, F3, F4, F5, F6. F1 to F5 hold. For F3 I checked `slices.Equal(nil, []string{})`, which is true, so rows with `wantMigrated: []string{}` pass against a nil dry-run `Migrated`. The F6 decision has one finding, below.

**Finding 1: MEDIUM**
- **Changed text:** `LackingTemplateBlocks` in `internal/runprofile/migrate.go` now appends `"<entry>.<from>: differs from <entry>.<to>"` for a differing former block. The reconcile walk's Warn in `internal/hubreconcile/walk.go:217-219` is unchanged.
- **Summary:** The F6 report goes through the Lacking hook, so the hub walk's Warn gives the wrong way forward for a differing pair.
- **Evidence:**
  - The Warn message reads "a seed-only config file lacks units its template holds, and reconcile never adds them".
  - Its `way_forward` reads `add each with "lym config profiles" from the hub's prime worktree, or leave it out on purpose`.
  - For `light.loom: differs from light.tapestry` the operator should remove or merge one of the two blocks. Adding does not apply, and leaving it out on purpose is not an option.
  - The same mismatch holds for the `lacking` doc in `internal/configsync/configsync.go:203`, "template units a seed-only module's file lacks". The same goes for the `lacking` comment in `reconcileEnvelope`, `internal/configcli/configcli.go:630`.
  - The new `configsync_test` row and the new `hubreconcile` test pin only the strings, so neither catches the mismatch.
- **Case:** After a mixed upgrade, an operator sees the Warn on every binary change. It tells them to add a unit that is already there.
- **Likely fixes:** Reword the Warn and its `way_forward` to cover both kinds of entry. Or give the differing pair its own result field.

**Minor, no severity:** In the F2 test, `t.Fatalf` after the plant and before the restore (the non-zero exit check) leaves the planted `profiles.yaml` in the board for the later subtests. This matters only when that test already fails.

Further tool calls: none. My context already held every file I needed.
