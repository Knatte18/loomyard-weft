```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests.** For each behaviour the diff adds or changes, this lists the test that pins it, or none.

| Behaviour | Pinning test | Result |
|---|---|---|
| `RenderRoleFile` writes the five on-demand files beside the role, drops their leading comment, and the role names each | `TestRenderRoleFile_WritesEachOnDemandFileBesideTheRoleThatNamesIt` (`prompt_test.go`) reads each file, checks it opens with `# `, and checks the role contains `` `<name>` ``. It breaks if a file is missing, still has its comment, or isn't named in the role. | Pinned |
| The role renders with the `decision_note_title` marker filled and no `command_index` | `TestRenderFiles_WriteWithoutLeadingComment` (`prompt_test.go`) checks for "Decision for <slug>:". `markers_test.go` now expects only `decision_note_title`. | Pinned |
| The role names `lym help index` | `TestRenderFiles_WriteWithoutLeadingComment` and `TestStart_NoStrandLaunchesAndSpawnsWatcher`. | Pinned (checks wording only) |
| `BeamUp` / `startOrch` renders the role file with no index argument | `TestStart_NoStrandLaunchesAndSpawnsWatcher` reads the rendered role. | Pinned |
| `ResumeContext` renders the role file for the non-verbatim cases only | `TestResumeContext_RendersPointerPerPhaseAndWritesMark` (`wantRoleFile` with the heading) and `TestResumeContext_FailureReturnsErrorAndWritesNoMark`. | Pinned |
| The watcher renders the role at a clear cycle, a compact cycle and an auto-reload, and a render failure returns to idle with a reason | `TestWatcher_ClearRendersRoleFileBeforeResume`, `TestCompact_CycleReloadsPluginsThenPointerNamingTheNote`, `TestWatcher_AutoCompactionReloadsPluginsThenPointer`, `TestWatcher_AutoCompactionReloadGoesIdleWhenTheRoleRenderFails` (reason prefix, no retry on a later tick), and the `role file` row of `TestWatcher_FailingRenderAbortsWithoutClear`. | Pinned |
| The `lym orch watch` verb passes no index source | Compile-time only. The index-source test was removed with `index.go`. | Pinned by compilation |
| `NewWatcher` and `BeamUpOptions` lose their index argument | Compile-time. Every call site was updated (`watchloop_test.go`, `smoke_cycle_test.go`, `beamcli/cli.go`). | Pinned by compilation |
| The on-demand files go through the shipped-stencil scans | `fixedStencilTexts` (`registry_test.go:23-29`) feeds `shippedStencils`, which the `roleskills`, `parentdirective`, `govalues` and `citation_enforcement` scans use. | Pinned |
| An on-demand file carries no marker | No row. `markers_test.go` lists only the role. A marker added to an on-demand file would reach `writeFilled` with a nil fill map. I did not check what `writeFilled` does with an unfilled marker. | None |
| `RenderRoleFile` writes the on-demand files before the role, so a role never names a file not yet there | No row. The prompt comment states this ordering as the reason for it. | None |
| The resume-context verb leaves a role file carrying the root's own index | The assertion was deleted along with the behaviour. | Removed with the behaviour, fine |
| `TestWatcherIndexSource` (removed) | Its subject, the strand-recorded `lym` binary lookup, no longer exists. `LymBin` is still used by reed. | Removed with the behaviour, fine |

## Findings

**F1 — LOW, class `design`.**
- Location: `internal/orchengine/prompt.go:72-78`, test at `internal/orchengine/prompt_test.go:74-107`.
- Summary: nothing pins that the on-demand files are written before the role, which the `RenderRoleFile` doc comment gives as its reason for the order.
- Evidence: the test only checks that all files exist afterwards. Swapping the order breaks no test.
- Suggested fix: either add an assertion that fails on the reversed order, or drop the claim from the doc comment.

**F2 — LOW, class `design`.**
- Location: `contracts/stencils/markers_test.go:36`.
- Summary: `TestStencils_TopLevelMarkers` lists `orch-role` only. The five on-demand files are said to carry no marker (`orchrole.go` comment, each file's header), but no row pins that they have none.
- Evidence: `registry_test.go:23-29` adds them to the shipped-stencil scans, but the scans I saw do not check markers. I did not read the marker scan itself, so confirm it before acting.
- Suggested fix: add a row per on-demand file with `nil` markers, or a loop over `orchrole.OnDemand` in the marker test.

No other behaviour the diff adds or changes lacks a test row. I raise no per-helper or duplicate-test finding. The new `TestRenderRoleFile_…` is the only added test and no other test covers its subject.

## Tool calls
1. `grep -n "shippedStencils\|fixedStencils" contracts/stencils/*_test.go`: to see which stencil scans consume the new on-demand texts, which the manifest did not show.
