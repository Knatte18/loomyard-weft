## Changes

- **F1** fixed as proposed: `TestWatchdogDaemon` passes the shared discovery loop a timing copy with `IdleCycles` 15, with a comment saying why.
- **F2** fixed as proposed:
  - `replayWeft` and `replayBolt` tag every classified set-aside commit before `cherryPickOnto`.
  - `replayBoardCommit` tags each lost-value or refused original as it reports it.
  - Every `replayBolt` error after the pick restores the starting tip through `ResetKeep(start)`, joining a restore failure to the cause.
  - `tagSetAsideOrRestore` is removed.
  - Rows added to `TestRecordsReplay`: a pre-existing set-aside tag at another sha (weft), and a failed by-entry apply (Bolt).
- **F3** fixed as proposed: `gitrepo.IsLocalChangeRefusal` is added, and `cherryPickOnto` wraps `ErrReplayDirty` only when it holds.
  Added a `TestRecordsReplay` row for a held `index.lock` and predicate assertions in `TestResetKeep`.
- **F4** fixed as proposed: `settleUnmovedWeft` returns a failed local read wrapped with the weft path.
  It still warns past `!hasUpstream` and an empty `behind`.
  Added `TestPull_WeftUpstreamUnreadableIsAnError`.
- **Self-check follow-ups:**
  - Docs of `ErrReplayDirty`, `replayWeft`, `replayBolt` and `doc.go` now state tag-before-move and the Bolt restore.
  - `restoreOnFailure` clears `Replayed` and `Dropped` after a successful restore.
  - The weft tagging row asserts an unchanged HEAD reflog, and the Bolt row asserts the by-entry error text.
  - The `gitrepo/reset.go` header names `IsLocalChangeRefusal`.
- **F5** is not changed; see Deferred.

## Disputed

None.

## Deferred

- **F5** (`decision`): whether the thirteen overview cuts are declared as PATTERN changes or main's text is restored on all but the fewest is a choice the review says to settle with `ly:orch`.
  This role sends no message about the review, so nothing was changed.

## Self-check

- fix-affected-set, LOW: `pullWeftArm`'s probe still warns past a failed upstream read.
  Not changed: the review named only `settleUnmovedWeft`, and the probe's warn-and-run-warp contract is documented; reversing it is an author decision, deferred with F5.
- fix-consistency, LOW: docs still said "nothing changed" — fixed.
- fix-consistency, LOW: `replayBolt` doc and `doc.go` omitted the restore — fixed.
- fix-consistency, LOW: `reset.go` header omitted `IsLocalChangeRefusal` — fixed.
- fix-consistency, LOW: `outcome.SetAside` carries tags after a stop — not changed; its readers (`logWeftReplay`, `logReplay`) run on success only, and the stop docs now say the tag stays.
- fix-proposal, MEDIUM: `IsLocalChangeRefusal` matches English stderr — fixed differently: not changed; `CherryPickAbort` in the same file already matches English stderr, and pinning the locale is a `gitexec`-wide decision outside this finding.
  Deferred with that reason.
- fix-proposal, MEDIUM: the Bolt restore is refused over files a failed apply wrote — not changed; the set-aside commits are tagged, and the doc and the joined error now say so.
- fix-proposal, LOW: tags on a refused or conflicting stop — docs fixed.
- fix-proposal, LOW: stale `Replayed` and `Dropped` after a restore — fixed.
- fix-code-correctness, MEDIUM: the weft tagging row passed with the defect — fixed with a reflog assertion.
- fix-code-correctness, LOW: docs "changing nothing" — fixed.
- fix-code-correctness, LOW: the Bolt row did not pin the error — fixed.
- fix-code-correctness, LOW: restore refused after writes, stale outcome — see the fix-proposal lines.
- fix-code-correctness, LOW: probe versus settle disagreement — see fix-affected-set.
- fix-code-correctness, LOW: no `IsLocalChangeRefusal` test — fixed in `TestResetKeep`.

## Gate repair

The verify run failed `TestNoDestructiveBypass_FabricengineProductionSource`: `replay.go` called `.ResetKeep(` for the Bolt restore, outside the destruction chokepoint.
The restore now goes through `restoreReplayTip` in `destroy.go`, which `replayBolt` calls.
`lyx gate test --tags integration` is green for `cmd/lyx` and `internal/fabricengine`; `internal/gitrepo` was green before the repair and untouched by it, and `internal/reedcli` passed in the verify run.
