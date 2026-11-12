# Fixer report — discussion, round 2

Every finding's premise held against the code.
All ten are fixed; none is disputed or deferred.

## Changes

- F1 (BLOCKING): fixed as proposed.
  D4 keeps only the exit record under `.lym/testcover/`.
  Each map's build lock, like its write lock, now comes from `lock.TrackedFileLockPath` with the `.build.lock` suffix, under `.lym/locks/`.
  The `PATTERN-lym-files` constraint bullet uses the proposed wording.
- F2 (MEDIUM): fixed as proposed, with one bound added.
  D9 registers every `lym gate testmap` call in a task under the slug, and in the prime only a `--upkeep` call registers under `runproc.HubOwner`.
  Only `--upkeep` runs the one-pass check.
  D9 now states the teardown bound, that the pending-records commit takes the maps.
  It also states the added bound: the check reads every `RoleTestMap` record of the owner, so in a task a hand call holds off a pass.
  The D9 criterion is reworded as proposed, and the `PATTERN-child-processes` constraint now covers every call.
- F3 (MEDIUM): fixed as proposed.
  Both "Decision for the parent" Open-risks lines are deleted, and D1's rationale cites "D1 stands".
  The support log's Q1 ledger line records the arbiter's confirmation.
- F4 (MEDIUM): fixed with option (b).
  In a task pair, D4 runs the run-scope origin reconcile through `fabricengine.ReconcileOrigin`.
  It spawns the push only when the verdict lets a loom run's push go, and otherwise pushes nothing.
  A D4 criterion is added, and the `PATTERN-owner-machine` constraint bullet is updated.
  Option (a) would have broken `PATTERN-pair-remote`'s rule that a task pair's records push carries its code branch.
- F5 (MEDIUM): fixed with the commit route.
  D5, retitled, has fabric's weft pull commit uncommitted `_lym/testcover/` paths through D4's scoped commit, under the records write lock, before the weft moves.
  It states the bound and the atomic-write premise, and a D5 criterion is added.
  The other route, a stop that names `lym gate testmap`, would stall prime syncs for the length of a pass.
- F6 (LOW): fixed as proposed in D7.
  A package job starts from `PlanUpkeep`'s in-memory map, never from the file on disk.
- F7 (LOW): fixed in D7.
  The redundancy step's age and drift describe the map's last write, and the step keeps them as they are.
  A reader that needs the map at HEAD rebuilds first, as the sweep does (D9).
- F8 (LOW): fixed as proposed in D11.
  The import takes each package's build lock without waiting, checks again, and skips a locked package, with the later-call bound.
- F9 (LOW): fixed as proposed in D1.
  The readers resolve through the anchor, and the writer works in the weft path fabric hands it.
- F10 (NIT): fixed as proposed in D11's rationale and in the support log's Rejected alternatives line.
- Support log: a Round 2 section under Review rounds records these fixes.

## Self-check

- fix-consistency, LOW, the D4 criterion said "no code branch" while D4 says "pushes nothing": fixed as proposed, and the criterion now says "pushes nothing".
- fix-consistency, LOW, D5's pull commit was not reconciled with the owner-only rule: fixed as proposed.
  D5 states that only the owner writes maps (D6, D11), so only its pull commits them.
- fix-consistency, LOW, D7's line naming the package job's guard as a planned-HEAD reader conflicted with the new start-from-plan sentence: fixed as proposed.
  The guard is dropped from the reader list, and the new sentence says the guard no longer applies to such a job.
- fix-consistency, LOW, D1 listed the audit among the anchor readers although it runs inside the writer: fixed as proposed, and the audit is now named as part of the writer.
- fix-consistency, LOW, the `PATTERN-owner-machine` bullet pointed to a PATTERN change that did not exist: fixed as proposed.
  The one `PATTERN-owner-machine` changes line now also says that a loom push, or `lym gate testmap`'s push in a task pair, reconciles run-scope origin first.
  A first attempt added this as a second PATTERN changes line, which the gate refused as a malformed action (`pattern-change-malformed`), so it was folded into the existing line.
- fix-way-forward, LOW, D9's refusal would misname a hand call as "the upkeep pass for the hub": fixed as proposed.
  D9 says the refusal names the registered call of the worktree by its pid.
- fix-claims, LOW, D5 overstated when the weft pull stops: fixed as proposed.
  The rationale now names a sync whose origin weft moved, and the bound says a tracked change "can still stop" the pull.
- fix-proposal: returned no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None reached this round.
