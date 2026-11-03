---
verdict: APPROVED
---

No finding.
Each focus question settles as follows.

## Rejected

- Unparseable `profiles.yaml` in the seed-only branch of `ReconcileHubWideAt`: `MigrateConfig` marks a parse failure invalid, the seed-only case assigns it to `reconcileErr` before any write of that module, and the common exit wraps it as a `*FileError` carrying `Module: "profiles"` and the file path (`internal/configsync/configsync.go`, the `m.SeedOnly` case and the `reconcileErr` exit).
  The row "profiles present that fails to parse is refused and left untouched" in `TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate` runs the dry run and the apply, asserts the `*FileError` and its module, and compares the file's bytes to the planted ones, so the path is reached and pinned.
- Lacking key of the reconcile envelope: `reconcileEnvelope` sets `lacking` only when `len(result.Lacking) > 0`.
  `TestConfigOverRealHub` plants a `light` profile without its `sampler` block, asserts the `profiles` entry carries `lacking` equal to `[light.sampler]`, asserts the `loom` entry carries no `lacking` key, and restores the file in a cleanup that also runs early through `sync.OnceFunc`.
- Former block beside a differing current block: `MigrateConfig` renames when no current block exists, deletes the former block only when `reflect.DeepEqual` holds, and otherwise continues without a line or an edit.
  `LackingTemplateBlocks` reports the differing pair as `<entry>.<from>: differs from <entry>.<to>`.
  The `MigrateConfig` doc comment, the `runprofile` `doc.go` paragraph, the second and third rows of `TestMigrateConfig`, the `TestLackingTemplateBlocks` row and the `configsync` row all state or pin that same rule, which is the parent's `Added after Discussion` decision.
  Decision text "reconcile reports the entry with both blocks" is met by the `lacking` line naming the pair.
- Test economy of the fix round: each added row or test pins a different surface of the same behavior (the hook function, the `ReconcileHubWideAt` result and file, the CLI envelope, the hub walk's Warn), and none repeats another's assertion.
  The `wantContains` entry of the differing row in `TestMigrateConfig` repeats what the unchanged-document check already asserts for a row with no `wantLines`; it adds no behavior and costs nothing, so it is not raised.
