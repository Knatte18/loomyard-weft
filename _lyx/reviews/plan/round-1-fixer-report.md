# Fixer report — plan, round 1

## Changes

- F1 (BLOCKING), fixed as proposed: card 1's `checkPrimeWritesHubWorkspace` asserts no `_portals` folder in the file, not on disk.
- F2 (BLOCKING), fixed as proposed: card 1's `hubWorkspaceFolders` item 3 skips an entry whose cleaned worktree path is not a direct child of `l.HubPath`, with a Debug log, and `TestRegenerateHubWorkspace` row 5 gains a third skip exit for a warp worktree added outside the hub.
- F3 (BLOCKING), fixed as proposed: card 5 edits `resolvePathOwnership` and uses `checkPathRequest`.
  Each emptied `_portals` chain directory passes `checkPathRequest` with `ownedUnderGeometryRoot` over the `_portals` container and `dirtinessNA`, with the hub as the container for `_portals` itself, before a non-recursive `removeContainedPath`.
  The Intent bounds the geometry-root set to the `_launchers` and `_portals` containers.
- F4 (MEDIUM), fixed as proposed: card 1 edits `internal/vscode/config.go`, with a Docs line for its Limitation paragraph, and both `go test` lines in `00-overview.md` run `./internal/vscode`.
- F5 (MEDIUM), fixed differently in one detail: every refusal or error from the sweep's removals, except the OS refusing a non-empty directory that holds a kept entry, becomes one detail and never stops the sweep or fails Reconcile.
  `TestReconcile_SweepsRetiredHubArtifacts` row 5 gains the escaping `_launchers/<AnchorRel>` case.
  The proposed list named `removePath`, which the F8 fix takes out of the sweep, so the list names `removeLink`, `checkPathRequest` and `removeContainedPath` instead.
- F6 (MEDIUM), fixed as proposed, plus one addition from the self-check: card 1 row 12 holds the lock, starts the regeneration in a goroutine, creates `b` with a raw `git worktree add`, releases, joins and asserts `b` is listed, and no pair is added from a goroutine.
- F7 (MEDIUM), fixed as proposed: card 3 drops the wiring-guard claim, and a failed mark read appends its raw read error to the Detail under `refusal-spec`'s transient exemption.
- F8 (LOW), fixed as proposed: card 5 removes the retired file through `checkPathRequest` followed by `removeContainedPath(launchersDir(l), path, false)`, recording `KindPathRemoved` `single`.
  `removePath` leaves card 5's Uses.
- F9 (LOW), fixed as proposed: the truncated-prefix sentence of `TestWriteHubWorkspace` is deleted.
- F10 (LOW), fixed as proposed: card 4 edits the surviving members of `portallauncher_test.go`, and its Intent rewords the file header to the launcher accessors and `_launchers` layout only.
- F11 (LOW), fixed differently in its cleanup: `checkPrimeWriteFailure` places a `0o444` file of different bytes at `HubWorkspacePath`, and `TestWriteHubWorkspace_RefusesEscapingSymlink` is named as the read-path refusal.
  The proposed read-only parent is dropped: the parent is the hub directory, a `0o444` file fails `root.WriteFile` on its own, and restoring only the parent left the file to fail the later prime-spawning steps of `TestSpawnScenario` (`internal/ideengine/spawn_scenario_integration_test.go:76-90`).
  The check's cleanup removes the file instead, and the check skips when `os.Geteuid() == 0`, as `internal/fabricengine/destructivegaps_integration_test.go:628` does.
- F12 (LOW), fixed as proposed: card 3's restore-failure row replaces `_launchers/<AnchorRel>` with an `fslink.CreateDirLink` to a directory outside the hub, and card 3 uses `internal/fslink#CreateDirLink`.
- F13 (LOW), fixed as proposed: card 2's row 5 is serial.

## Self-check

- fix-affected-set, LOW, `TestRemoveLaunchers_PreservesForeignContent`'s doc comment still names the portal half of the teardown: fixed as proposed, and card 4's Intent rewords that doc comment.
- fix-consistency, LOW, card 5's opening sweep sentence still says it removes only through the executors: fixed as proposed, and the sentence names `removeLink`, or `removeContainedPath` after `checkPathRequest`.
- fix-consistency, LOW, card 5's ImpactSummary says no other caller changes, although the geometry-root set grows: fixed as proposed, and the summary names the grown set and the reworded one-member comment.
- fix-claims, MEDIUM, `checkPrimeWriteFailure`'s cleanup leaves the `0o444` file, which fails later prime spawns: fixed with own change, the cleanup removes the file (see F11).
- fix-proposal, MEDIUM, the same finding: merged with the fix-claims finding above.
- fix-proposal, LOW, the F11 mode-based arrangement does not refuse for root on POSIX: fixed with own change, the check skips under `os.Geteuid() == 0`, because the write branch has no platform-neutral trigger: a directory or escaping link at the path fails at the read first.
- fix-proposal, LOW, row 12 detects a missing lock only when the goroutine reaches the lock before `b` exists: fixed with own change.
  Card 1 creates `hubWorkspaceBeforeLockHook` with `SetHubWorkspaceBeforeLockHookForTest`, which runs just before the lock is taken, as `addFailAfterWarpHook` does for Add.
  Row 12 waits for the hook's signal before creating `b`, and is serial.

## Disputed

None.

## Deferred

None.
