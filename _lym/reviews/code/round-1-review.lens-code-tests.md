```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. I mapped each behaviour the diff adds or changes to the test row that pins it.

**Behaviour → pinning test**

| Behaviour | Pinning test |
|---|---|
| Sweep removes fabric-written scripts and `.code-workspace` files, then the emptied directories, container last | `fabricengine.TestSweepRetiredLaunchers` rows `OnlyFabricWrittenFilesUnderNestedAnchors` and `DryRunListsTheRemovalsAndRemovesNothing` (`retiredlaunchers_test.go:71`, `:111`) |
| Sweep keeps foreign files and the directories above them | `ForeignFileKeepsItsAncestors` (`:79`) |
| Sweep never follows a symlink inside the container | `SymlinkIsKeptAndNotFollowed`, which also asserts the outside sentinel survives (`:101`, `:186`) |
| Sweep of a hub with no `_launchers` is empty | `NoContainer` (`:55`) |
| `_launchers` is a regular file: kept, not removed | `ContainerIsAFile` (`:63`) |
| `_launchers` is a symlink or junction | **none** (see finding 1) |
| `.cmd` script names are swept | **none** (finding 2) |
| A removal failure stops the sweep and returns the partial result with the error | **none** (finding 3) |
| `SweepRetiredLaunchers` fails safely on an unreadable hub (`OpenRoot` error) | **none**, and the code is trivial |
| Dry run lists and removes nothing, with an empty mutation record | `DryRunListsTheRemovalsAndRemovesNothing` |
| Applied sweep records `KindPathRemoved` entries | the mutation-record assertion in `TestSweepRetiredLaunchers` (`:166`) |
| `lym cleanup` has a `launchers` segment, last in order | `cleanupcli.TestPrimeSegments` (`cli_test.go:71`) and the integration test (`cleanup_integration_test.go:200`) |
| `lym cleanup` dry run lists the two scripts, the pair directory, then `_launchers` last | `TestCleanupVerb_…` (`:206`) |
| `lym cleanup --apply` removes `_launchers` once; the dry run leaves it | `assertLeftoversRemain` and the post-apply check (`:302`) |
| `lym cleanup` refuses outside the prime | existing refusal step in the same integration test |
| The `kept` output of the cleanup segment | **none** at CLI level; the engine test covers `Kept` |
| `Add` writes no launchers and its rollback no longer touches them | `hubworkspace_test.go:83,109,128` (`assertNoHubContainers`), plus the rewritten `TestAddRollback_*` and `livestate` Add case through `failAddBeforeWiring` |
| `EnsureAnchor` and `AnchorChange` no longer write or move launchers | `TestAnchorChange` (the launcher assertions were dropped). `assertNoHubContainers` is not called there. |
| `Remove` and `Prune` no longer tear down launchers; `PairParts` drops the `Launchers` field; `refuseWhenNothingRemains` loses its launcher branch | `livestate` Remove and Prune cases, `TestRemove_UnreachableOriginFailsClosed`, `TestPairParts` (`pairparts_integration_test.go`) |
| A pair whose only local remnant was a launcher directory no longer counts as present | none seen. This used to be reachable through `refuseWhenNothingRemains`, but it is only a removed branch. |
| `launchers_restored` reconcile action removed | `TestReconcile_RestoresDeletedLaunchers` was deleted. No test asserts the verb no longer restores a missing directory, which is acceptable. |
| `_launchers` stays reserved | `junctionnames_test.go:76-87` and `slug_test.go:33` |
| `ownedUnderGeometryRoot` still accepts a `_launchers` root | `destroy_test.go:629` and `destroy_unix_test.go:27,118` |
| `retiredHubWorkspacePath` / `sweepRetiredHubWorkspace` after the `launchersDir(hubPath)` signature change | `TestSweepRetiredHubArtifacts_UnresolvablePrime` (`portals_test.go`) and `portals_sweep_integration_test.go` |

**Findings**

1. **MEDIUM, design**, `internal/fabricengine/retiredlaunchers_test.go:19`. No row seeds a symlink or junction at `_launchers` itself.
   - The code relies on `root.Lstat` plus `!IsDir()` to keep it (`retiredlaunchers.go:69-79`). That is the focus directive's "symlink at `_launchers`" case.
   - Swapping `Lstat` for `Stat` would make the sweep descend through the link, and no test would fail.
   - The existing `SymlinkIsKeptAndNotFollowed` row only covers a link below the container.
   - Add a row with `linkRel: "_launchers"`. Its expectations are `Kept` naming `_launchers`, and the outside sentinel and the link both surviving.

2. **LOW, design**, `retiredlaunchers.go:22`. Every `.cmd` name in `retiredLauncherFileNames` (`ide.cmd`, `run.cmd`, `fabric-checkout.cmd`, `ide-menu.cmd`) is untested; the rows seed only `.sh`.
   - Dropping the `.cmd` entries fails nothing.
   - Add one `.cmd` file to a row's seed tree, as part of an existing row.

3. **LOW, design**, `retiredlaunchers.go:99-105`. The error path of a failing removal in apply mode is untested.
   - That path returns the partial `Removed` list and the error, and the cleanup segment surfaces it.
   - A fake is awkward, since it needs a non-empty directory race or a read-only parent. Weigh this as an optional gap.

4. **MEDIUM, design**, deleted `internal/fabriccli/hubmutation_integration_test.go` row `Remove_RefusesDriftedLauncherDirLinkWithRefusalObject`.
   - The row pinned behaviour that still exists: the `fabric remove` failure envelope carries a `refusal` object with non-empty `check`, `what`, `target` and `reason`, `check == CheckContainment`, a flattened `error`, and a `mutations` key.
   - A grep of `internal/fabriccli` and `internal/battencli` tests for `"refusal"` finds only absence checks: `envelope_test.go:143` and `cleanupremove_integration_test.go:408`.
   - The positive refusal-object shape through the CLI now has no test.
   - Another real `Remove` containment refusal should carry the row. A pair whose warp worktree path is a link out of the hub is one candidate.
   - I did not check whether a surviving test covers `RefusalOf` at the engine level.

5. **LOW, design**, deleted `TestRemoveLaunchers_ContainmentRefusalSurvivesSurfaceRefusal` and `TestRefuseUncontainedPath` (`ancestors_test.go`).
   - They pinned that a gate refusal propagates through `surfaceRefusal`, and the `containmentFailure` refusal wording and `..` handling.
   - `containmentFailure`, `resolveAncestorSymlinks` and `containmentPath` survive in `ancestors.go`, yet no `_test.go` file references any of them now (the grep returned nothing).
   - They are covered only indirectly, through the gate in `destroy_test.go` and `destroy_unix_test.go`.
   - Check that those tests cover the `..` and escaping-ancestor cases. Check the remaining call sites of `surfaceRefusal` as well: the launcher callers in `Remove` and `Prune` are gone, so any test of it that survives is now unreachable behaviour.

6. **LOW, consistency (test economy)**, `internal/fabricengine/anchorchange_integration_test.go:207-240`. The test asserts neither "no `_launchers` appears at the new or old anchor" nor the wiring and workspace change that remains.
   - `assertNoHubContainers` is not called there; it runs only in the `hubworkspace_test.go` tests.
   - Add one call to the existing `TestAnchorChange` subtest. A new test is not needed.

7. **Not a defect**:
   - The new `launchers` count of 4 in `cleanup_integration_test.go:206` is a literal tied to the seeded layout (two scripts, the `gone-pair` directory, the container). That is fine, since it is a derived fixture, not a pinned tally.
   - `TestSweepRetiredLaunchers` is correctly unique per the redundancy report, so no economy finding applies.
   - `TestAddRollback_*` and the `livestate` Add case replaced the launcher blocker with the `SetBeforeWireForTest` hook, so the rollback paths are still exercised.

**Tool calls**
1. `git diff` of the changed test files (cleanup, rollback, livestate, anchorchange, portals, slug, remove_archive): the manifest had only non-test diffs.
2. `grep` for `HubReservedNames`/`IsReservedHubName` and `SweepRetiredLaunchers`/`"launchers"` tests: to confirm reserved-name and cleanup coverage.
3. `git show` of the base versions of `ancestors_test.go`, `reconcile_stale_registration_test.go` and `destroy_containment_toctou_integration_test.go`, plus the `hubmutation` diff, and a grep of surviving tests for `containmentFailure`: to see what the deleted tests pinned.
4. `ls` and `git diff` of the stale-registration and toctou deletions, a grep for `"refusal"` in the fabriccli/battencli tests, a grep for `assertNoHubContainers`, and the base `portallauncher_test.go` test names: to find the replacement coverage for each deleted test.
