```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Per behaviour, the pinning test or none**
- **Update at base value, comments kept:** `TestFollowTemplate` row 1 (`internal/yamlengine/follow_test.go:26`).
- **Template drop removes the unit, including a mapping the removal empties:** `TestFollowTemplate` rows 2 and 3.
- **Template-added subtree or key, including into a flow mapping:** `TestFollowTemplate` rows 4 and 5.
- **Operator value kept, with and without a template move (`Differs`):** `TestFollowTemplate` rows 6 and 7.
- **Operator removal kept:** `TestFollowTemplate` row 8.
- **No-base rules (add lacking, keep and report differing, keep extra silently):** `TestFollowTemplate` row 9, plus `TestReconcileHubWideAt_SeedOnlyFileFollowsItsTemplate` (`internal/configsync/configsync_test.go`), the "with no base" row.
- **Scalar turned into a mapping, empty file, non-mapping document:** `TestFollowTemplate` row 10, row 11 and its closing error check.
- **A sequence compared as one unit:** none. No `FollowTemplate` row uses a list, although the doc comment and the focus directive name it.
- **Nested `Differs` or `Updated` paths:** the dotted path is only exercised through `light.darn.loom.*` in the configsync test, so it is pinned.
- **Template base written when absent or stale, nothing written on a dry run, second apply a no-op:** `TestReconcileHubWideAt_SeedOnlyFileFollowsItsTemplate`. It also asserts `Applied` and `WroteTemplateBase`.
- **Base written when the file is seeded from the template, prime or legacy:** `TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate`. Every `profiles` row asserts `WroteTemplateBase` after the apply.
- **Prime-copy and present `models` file gain the template aliases they lack:** the two reworded `models` rows of that same test.
- **`profiles` file with no base gains the template's units, with `Unsettled` still listed:** the reworded `profiles` rows of that same test.
- **Unreadable base (`readTemplateBase` `FileError`), or a base that fails to parse:** none.
- **Hub file left untouched when the base read or merge fails:** none. The `wantFileError` row covers only an unparseable hub file.
- **`UnsettledFormerBlocks`:** `TestUnsettledFormerBlocks`, including the unparseable case.
- **Light drops `code_reviewer_fan`, `docs` and `speed` keep it:** `TestTemplateLadder` (the `light` row now has no solo recipes) and the two `battencli` run-settings rows. Whether the ladder table has a `docs` row is not in my context.
- **Walk carries the change, keeps the tuned value, commits the file and base, mails the orch:** `TestEnsure_CarriesATemplateChangeIntoASeedOnlyConfigFile`. It asserts only the `removed` mail line.
- **Mail lines for `added`, `updated` and "kept the hub's" (`Differs`), no mail with no lines, and no lines for a non-`SeedHub` result:** none. `followedLines` and `notifyTemplateFollowed` are only reached through the `removed` case.
- **Rollback of a written base on a failed walk (`snapshotConfig`), or a failed `--apply` (`snapshotBoardConfig`, `restoreBoardConfig`):** none. The existing restore tests assert module files, not `*.template-base`.
- **`configcli` dry-run envelope: `removed` listed and nothing written:** `TestConfigOverRealHub` (`internal/configcli/configcli_integration_test.go`).
- **`configcli` envelope keys `updated`, `differs` and `unsettled`:** none. The test only checks that `loom` has no `differs`, so a broken or misnamed key would not fail it.
- **`--apply` committing the base through `ChangedFilesRel` (`configcli/hubwide.go`):** none. The test only dry-runs.
- **`lym fabric reconcile` committing the base (`fabric.go`):** none.
- **Clone's `KindFileWritten` record for the base (`fabriccli/clone.go:66`):** none.
- **`ChangedFilesRel` itself:** reached only through the walk test, so the `Applied`-only, `WroteTemplateBase`-only and legacy-file branches are not pinned directly.

**Findings**
- F1, MEDIUM, design (no test). `internal/configcli/configcli_integration_test.go`, `internal/fabriccli/clone.go:66`, `internal/fabriccli/fabric.go:716-719`, `internal/configcli/hubwide.go:223`: the committing of the template base by `--apply`, `fabric reconcile` and clone has no test. Record rule 5 and the "committed on Bolt with the file" requirement hold for these three callers only through `ChangedFilesRel` and the clone `rec.Append`, and a regression there would leave an untracked base. Fix: add an `--apply` assertion that the base is in the Bolt commit, and a `fabric reconcile` or clone assertion.
- F2, MEDIUM, design (no test). `internal/hubreconcile/walk.go:400-433`: only the `removed` mail line is asserted. The `added`, `updated` and "kept the hub's" lines, and the no-mail case, are not. Record rule 5 names `differs` and `updated` reporting as the point of the change. Fix: add rows for an `updated` line, a `Differs` line with both values, and no mail when nothing followed.
- F3, MEDIUM, design (no test). `internal/configcli/configcli.go:629-641`: the envelope keys `updated`, `differs` and `unsettled` are asserted nowhere. Rule 5 requires the dry run to show `updated` and `differs`. Fix: extend the `TestConfigOverRealHub` profiles scenario to plant an operator-set value and a former-name pair, and assert both values appear under `differs` and the pair under `unsettled`.
- F4, LOW, design (no test). `internal/configsync/configsync.go:244-254,1022`, `walk.go:471-476`: an unreadable or unparseable base, and the restore of a written base on a failed pass, are untested. The restore matters because a base written before a later failure would otherwise be committed or kept stale. Fix: add a row with a corrupt base asserting a `FileError` and an untouched hub file, and a walk failure test via `afterHubWideWrite` asserting the base is restored or removed.
- F5, LOW, design (no test). `internal/yamlengine/follow_test.go`: no row has a sequence as a unit. Fix: add a row where a list at the base's value takes the template's new list, and one where an operator-edited list is kept.
- F6, LOW, scope (test economy). `internal/configsync/configsync_test.go`, the rows "a key the operator set is kept and listed", "a key the template added since the base is added" and "a key the operator removed stays removed": they repeat `TestFollowTemplate` rows 6, 4/5 and 8 at the reconcile layer. The `removed` row and the base and dry-run assertions are unique to the configsync test. Fix: drop or merge the three rows into the one case that exercises the file and base plumbing.

Outside my lens: in `configsync_test.go` the new test `TestReconcileHubWideAt_SeedOnlyFileFollowsItsTemplate` was inserted directly after the doc comment of `TestReconcileHubWideAt_MigratesLegacyFabricConfig`. That test now has no doc comment, and its old comment text runs into the new test's comment (around lines 910-1091).

**Tool calls:** none. My context already held every diff and test body.
