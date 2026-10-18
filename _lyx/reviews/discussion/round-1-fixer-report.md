# Fixer report — discussion round 1

Every finding's premise held against the code; all 18 are fixed in `_lyx/discussion/decision-record.md`, with the support log updated to match.

## Changes

- F1 — fixed as proposed: D4 reads command keys over the template and board default layer only (`configengine.Dirs` with only `Board`), switch keys over the task worktree's stack; Bound rewritten to name the layers and why no task branch, adopted branch or board entry reaches them; criterion added that a command key in the task worktree's tracked `batten.yaml` is not run.
- F2 — fixed as proposed (first option): a per-slug teardown lock under `fabricengine.HubScratchDir`, children shared, `Remove` exclusive, released after the worktree teardown; both platforms named in the criterion.
- F3 — fixed differently: the proposal's example of a warp-containment guard was not made, since `deleteWarpBranch` already keeps a warp branch with unlanded work while `Remove` succeeds (`remove.go:126-131`), so such a guard would only add a refusal for work that is never lost. The weft rule is explicit instead: a local pair's records are disposable and dropped, with the dropped weft tip recorded. No new refusal remains, so no flag answers one. Criterion now covers a pair that ran a shed run.
- F4 — fixed as proposed: every child checks its worktrees exist before creating the lock directory, under the teardown lock when it has a slug; criterion adds the re-add.
- F5 — fixed as proposed: current blocking behaviour stated; try-lock plus pending flags that every holder re-checks after release; the clause leaving it to the plan is gone.
- F6 — fixed as proposed: `Remove`'s wait defaults to `StatusPushLockWait`, with a refusal and its way forward; criterion added.
- F7 — fixed as proposed: `Commit` returns whether a commit landed and the core spawns only then.
- F8 — fixed as proposed: a hidden slug flag the spawn entry point passes and the child's gate install honours.
- F9 — fixed as proposed (first option): `--force` answers dirtiness only and the warp branch keeps its existing gate; the new rule covers the weft alone.
- F10 — fixed differently: no guard remains to order, so a local remove commits no pending records and discards them with the weft, rather than counting them toward a guard.
- F11 — fixed as proposed: `archiveWeftTip` and `deleteRemoteBranch` check the mark; `lyx fabric cleanup` named; criterion added.
- F12 — fixed as proposed: lazy per-key-class resolution, reconciled with F1.
- F13 — fixed as proposed: the criterion measures beyond the status commit and the process.
- F14 — fixed differently: rather than wrapping the `Setting` lookup so `open_window` reads true, batten tells the producer the rows whose switch the run forces, and the Warn keys on that told fact; this keeps the lookup a plain config read.
- F15 — fixed as proposed (refuse option): a `%` value on Windows runs no process and warns with a way forward for that placeholder's source; criterion adds `%`.
- F16 — fixed as proposed: either surviving mark makes a pair local; no surviving mark is ordinary.
- F17 — fixed as proposed: the spawn entry point checks the mark and records the skip on its caller's result; the child only logs.
- F18 — fixed as proposed: the `row-entry-gate` bullet moved to the support log's Interview, the `pair-from-parent` bullet deleted.

## Self-check

- fix-consistency, MEDIUM, existence check limited to children with a slug — fixed: every child checks, and a pair push always carries its slug.
- fix-consistency, MEDIUM, new refusal's place against `RemoveRefusal` and the existing refusals — fixed: the lock is taken after every existing refusal, and `pairteardown` gets the refusal from `Remove`.
- fix-consistency, LOW, Notes for the plan writer contradict D3 — fixed.
- fix-consistency, LOW, Scope item 3 and PATTERN-mutation-record omit cleanup — fixed.
- fix-consistency, LOW, `StatusPushLockWait` sentence stale — fixed.
- fix-proposal, MEDIUM, a records-only holder clears a code-side flag — fixed: flags per side, and a holder spawns the push of a side it does not push.
- fix-proposal, MEDIUM, a paused child makes `Remove` refuse as if stuck — fixed: the refusal names a pause as a cause and resuming it as a way forward.
- fix-proposal + fix-way-forward (merged), MEDIUM, a plain `lyx config set` writes a layer batten ignores — fixed: `--layer default` named, and the row warns on an ignored value.
- fix-proposal, MEDIUM, a plain add adopting a kept marked branch stays local — fixed: a plain add clears the mark.
- fix-way-forward + fix-claims (merged), MEDIUM, the `%` way forward fits only `{task_dir}` — fixed: per-placeholder source.
- fix-way-forward, MEDIUM, Worktree-Teardown row's handling of the refusal unstated — fixed: the row stands Stuck with `lyx batten run <slug>`.
- fix-way-forward, LOW, the refusal's test waits 30 s — fixed: the wait is shortened by tests.
- fix-way-forward, LOW, a child exiting before a `Remove` that then fails loses its push — fixed: the lock is taken after the refusals, and a failing `Remove` spawns the pair's push after releasing.
- fix-claims, MEDIUM, pending records make the weft worktree removal refuse without `--force` — fixed: the weft gate takes a records-disposable kind on the local path.
- fix-claims, LOW, "no quoting stops it" overstated — fixed: scoped to inside double quotes, in the record and the support log.

## Disputed

None.

## Deferred

None.
