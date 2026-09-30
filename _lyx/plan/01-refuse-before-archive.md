# Card 1 — Move Remove's dirtiness refusals ahead of the archive and every teardown

**Edit:**
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine/remove.go`
- `internal/fabricengine/doc.go`
- `internal/fabricengine/remove_archive_integration_test.go`
- `internal/fabricengine/remove_refusal_remedy_integration_test.go`
- `internal/fabricengine_test#TestRemove_RefusalNamesStrandedPortalTeardown`
- `internal/fabricengine/mutation_record_integration_test.go`
- `internal/fabricengine_test#TestMutationRecord_RemoveDirtyWarpRefusalCarriesThePortalAndLauncherDeletions`
- `internal/fabricengine_test#removeCase`
- `internal/fabricengine/livestate_doc_test.go`

**Delete:**
- `internal/fabricengine#nameStrandedPortalTeardown`
- `internal/fabricengine_test#TestRemove_ResumedRemoveReusesArchiveTag`
- `internal/fabricengine_test#TestRemove_RefusalWithNothingStrandedOmitsRemedy`
- `internal/fabricengine_test#portalAndLauncherPermittedRoots`

**Uses:**
- `internal/fabricengine#refuseDirtyWeftWorktree`

**Intent:**
A no-force `Remove` refused for dirtiness today has already pushed an `archive/<slug>/<tip>` tag that misses the uncommitted records and torn down the portal and launchers.
Reorder `Topology.Remove` so every refusal precedes the archive and the archive precedes every removal:
slug validation → prime refusal → target-exists check → origin-record read → merge-in-progress refusals → no-force warp dirtiness check → no-force weft dirtiness check (`refuseDirtyWeftWorktree`) → `archiveWeftTip` → `removePortal`/`removeLaunchers` → junction sweep → warp worktree removal → weft teardown → warp branch deletion.
`force` still skips only the two dirtiness checks, never the archive.
A failed status probe (`check warp worktree status`, `check weft worktree status`) now refuses with no tag pushed and nothing torn down.

Production changes in `remove.go`:
- The dirty refusals return their errors directly.
  The base texts stay byte-identical (`worktree has uncommitted changes; use --force`, the weft `siblingDirtyRefusal` text wrapping `ErrPairSiblingDirty`, and both `check … worktree status` wrappers), since `battencli` matches the sentinels and the livestate matrix matches the warp text by substring.
- Delete `nameStrandedPortalTeardown` and the `archivedEntries` local; with nothing recorded before the refusals its remedy branch can never fire.
- Rewrite the file header and the `Remove` doc comment for the new order.
  Drop the stale claim that the portal teardown runs early "so the teardown still runs when the worktree dir itself is already gone": a missing worktree dir already returns "worktree not found" at the target-exists check.
  State that a refusal, the dirtiness checks and their probes included, leaves the hub and the weft origin untouched, and that a failure after the archive leaves the tag for the re-run to reuse.
- Update the `archiveWeftTip` call-site comment to say it runs after every refusal.

`doc.go`: the `Remove` passage ("After its refusals and before its first mutation …") names the dirtiness checks among the refusals the archive follows.

Test changes, all in `internal/fabricengine` and behind `//go:build integration`:
- `remove_archive_integration_test.go`: delete `TestRemove_ResumedRemoveReusesArchiveTag`, which reached `archiveWeftTip`'s reuse path through the refused-after-archive state the reorder removes, and replace it with two tests.
  `TestRemove_ReusesSameTipArchiveTag` keeps that reuse path covered: it builds a pair (`Add` without `SkipPush`), commits a record with `commitLyxFileInWeft`, then plants the state an archive leaves when a later teardown step failed by creating `archive/<slug>/<tip[:12]>` at the tip in the weft repo (`mustWeftRepoRoot`) and pushing it to origin with `gitkit.MustRun`.
  A no-force `Remove` then succeeds with `ArchiveTag` naming the planted tag, `tagTargetAt(fixture.WeftBare, tag)` at the tip, and `archiveTagsAt(fixture.WeftBare)` listing exactly that one tag.
  `TestRemove_WeftDirtyRefusalPushesNoTagAndResumeArchivesOnce` covers the refuse-then-resume flow.
  It dirties the weft worktree with an untracked file `git status` reports (the shape `TestRemove_UntrackedDriveReportRefusesWithSiblingDirty` uses), asserts a no-force `Remove` fails with `errors.Is(err, fabricengine.ErrPairSiblingDirty)` and that `archiveTagsAt` is empty on both `fixture.WeftBare` and the local weft repo.
  It then commits that file in the weft worktree (`git add -f`, `git commit`), captures the new tip, re-runs a no-force `Remove`, and asserts success with `archiveTagsAt(fixture.WeftBare)` equal to exactly `archive/<slug>/<tip[:12]>`, pointing at the post-commit tip.
  Reword the file header's "before any mutation" and "resumed reuse" phrasing to match the new order and the two new tests.
- `remove_refusal_remedy_integration_test.go`: rename `TestRemove_RefusalNamesStrandedPortalTeardown` to `TestRemove_DirtyRefusalLeavesPairIntact` and invert it.
  After the tracked-warp-dirt refusal, assert the error contains `uncommitted changes` and does not contain `lyx fabric reconcile`; the portal link, launcher dir, both worktrees, the warp branch and the weft branch (`branchExistsAt`) all exist; `res.Mutated().Len()` is 0; and no `archive/<slug>/` tag exists on the weft origin or in the local weft repo.
  Drop the `Reconcile` remedy half.
  Delete `TestRemove_RefusalWithNothingStrandedOmitsRemedy`; the rewritten test subsumes it.
  Keep `TestRemove_StatusFailureNamesPathAndCommandOnce` unchanged.
  Add `TestRemove_WarpStatusProbeFailurePushesNoTag`: on a real pair, overwrite the warp worktree's `.git` gitfile with a `gitdir:` pointing at a missing directory so `git status` fails, run a no-force `Remove`, and assert the error contains `check warp worktree status`, no `archive/<slug>/` tag exists on origin or locally, and the portal, launchers and weft worktree still exist.
  Rewrite the file header: it now pins that a no-force refusal, dirtiness or failed probe, leaves the pair and the weft origin untouched.
- `mutation_record_integration_test.go`: rename `TestMutationRecord_RemoveDirtyWarpRefusalCarriesThePortalAndLauncherDeletions` to `TestMutationRecord_RemoveDirtyWarpRefusalRecordsNothing`.
  It keeps the `RefusalOf(err)` false assertion and replaces the link/path-removed checks with `res.Mutated().Len() == 0`.
  The file header and the test's doc comment stop calling the `Remove` refusal a mutated-then-errored path; the Add rollback stays the file's mutated-then-errored case.
- `livestate_verbs_test.go`: `removeCase`'s dirty cells keep `KindRefusedBefore` and `Substring: "uncommitted changes"`, drop `PermittedRoots`, and their `Effect` asserts the portal link, launcher dir and warp worktree still exist (`assertExists`).
  Rewrite `removeCase`'s doc comment and the in-switch comment, which cite the old line numbers and order.
  Delete `portalAndLauncherPermittedRoots`, which has no remaining caller.
- `livestate_doc_test.go`: delete the "One known refusal-with-side-effects anomaly" section, since a dirty `Remove` refusal now has no side effects.
  The numbered mutation-test history entries below it record past runs and stay as written.

`CONSTRAINTS.md`, `docs/overview.md`, `archive.go`, `cleanup.go` and `battencli`'s `teardownRefusal` stay unchanged.

**ImpactSummary:** `Topology.Remove`'s step order and refusal errors (reconcile suffix gone); `battencli` sees the same sentinels and base texts; every change beyond `remove.go` is a comment or a test in `internal/fabricengine`.

**Commit:** 1: fabric: refuse a dirty Remove before archiving or tearing anything down
