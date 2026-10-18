# Round 2 fixer-report — discussion

## Changes

- F1 — fixed as proposed, carried out as two hidden flags, one per role: D1 now has a pause slug and a teardown slug, and the teardown slug is set only for a push of the pair's own worktrees, so prime records-side pushes (batten's seam, `lyx batten cleanup`) take no teardown lock.
  The lock sentences use "teardown slug", and the Teardown criterion now says the Worktree-Teardown status push does not delay or refuse that row's `Remove`.
- F2 — fixed differently.
  The proposal set the child's working directory to the hub directory and told it a log directory.
  The bare hub is not a lyx worktree: root's pre-run (`cmd/lyx/main.go`) arms the logger sink and runs `tendercli.InstallNetworkGate` from the working directory before any fabric flag is read, and that install returns without installing a gate outside a hub (`internal/tendercli/gate.go`).
  So D1 now starts the child in the hub's `_board` worktree, which is outside the pair and still resolves the hub, and has the gate install read the pause slug flag.
  The sink's cwd-anchored resolution then lands in `_board/.lyx/logs` (`fabricengine.HubLogsDir`) with no flag-order dependence.
  The Windows sentence now covers the lock file, the working directory and the log file.
- F3 — fixed as proposed: D4 makes the bound the producer's contract, reading the command through an executable-key lookup the owner tells, over the template and board default layer only.
  A row whose owner tells none refuses before running anything, with its way forward.
  The Bound, the `PATTERN-command-row` text, the refusal-way-forward constraint and a new acceptance criterion carry the change, and batten's lookup text was reworded to that lookup.
- F4 — fixed as proposed (Constraints, destruction chokepoint).
- F5 — fixed as proposed: D1 names `fabric.push.lock`'s acquisition, through both `coalescePush` and `acquirePushLock`, as the change site, and says `Bolt.Sync` over `board.push.lock` is unchanged.
- F6 — fixed as proposed: D3's remove bound states the hand-set-key case, and remove reports the local path and the marked branch; the local-remove acceptance criterion asserts that report.
- F7 — fixed as proposed (D4 Bound), extended by the self-check to name the missing-lookup refusal.
- F8 — fixed as proposed, with the self-check's correction: the bound says a hand-run child without the teardown slug can race `Remove` and recreate its push lock's directory, since its existence check holds no lock.
- Support log: added a Round 2 section and updated Q10.

## Self-check

- fix-way-forward, LOW: the missing-lookup refusal's way forward ended at a code change, and the Bound omitted that it stops the run. Fixed as proposed: the refusal now adds a re-run on the rebuilt `lyx`, and the Bound names it.
- fix-way-forward, LOW: the child's Warns move to `HubLogsDir` and no reader is told. Fixed as proposed: Open risks names that directory.
- fix-consistency, MEDIUM: the told executable-key lookup departs from `PATTERN-recipe-setting-lookup` (`PATTERN.md:49`). Fixed as proposed: a "changes" line for that entry under PATTERN changes.
- fix-consistency, LOW: the Bound omitted the missing-lookup stop. A duplicate of the first way-forward finding, already fixed.
- fix-consistency, LOW: the Teardown criterion's "in-flight push finishes first" disagreed with the prime-push carve-out. Fixed as proposed: it now names the pair's in-flight push (loom's or darn's).
- fix-consistency, LOW: respawns did not say they pass the slugs. Fixed as proposed: the F8 bound says every respawn goes through the spawn entry point with the slugs of the push it serves.
- fix-consistency, LOW: support log Q10 was stale. Fixed as proposed.
- fix-claims, MEDIUM: a hub-directory working directory installs no pause gate. Fixed differently: see F2 above; the child runs in `_board`, not the told `--warp-path` resolution the fork suggested, because root's pre-run reads the working directory before fabric's flags.
- fix-proposal, MEDIUM: the same gate defect. A duplicate, fixed under fix-claims.
- fix-proposal, MEDIUM: a told log directory cannot take effect before root's pre-run arms the sink. Fixed differently: the told log directory is dropped, and the `_board` working directory sends the cwd-anchored sink to `HubLogsDir`; D1 states that these logs are not archived with the pair's.
- fix-proposal, MEDIUM: F8's claim that the existence check prevents writes at a removed path does not hold without the lock. Fixed as proposed: see F8.
- fix-proposal, LOW: the Bound omitted the missing-lookup stop. A duplicate, already fixed.
- fix-proposal, LOW: a holder's respawn did not say it carries the teardown slug. A duplicate of the fix-consistency respawn finding, already fixed.

## Disputed

None.

## Deferred

None.
