```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked each bound, guard, ordering promise and claim about existing code in `decision-record.md`.

**Claims attacked**

1. D1 step 3, "calls `Add` with the run's parent and anchor": **broken**. `Add` refuses any non-empty anchor on a live weft.
2. D1 bound, "never re-creates a pair whose weft tip an archive tag covers": **broken**. A local weft branch reads as live whatever the tags say.
3. D1 bound, `AdoptLocal` takes "the branch the run itself made": held. A new run cannot pass Worktree-Create while an unmarked warp branch exists (`add.go:291`).
4. D1 bound, "nor one whose records worktree directory still stands": held (`add.go:329`).
5. D1, a continue pass with the task worktree absent is refused and dobby reads that as a full pass owed: held (owner-machine background, Passes section).
6. D2 split between permanent and transient answers: **broken**. `Add`'s errors carry no sentinel for transport or probe failures, and operational refusals would be read as permanent.
7. D2, a `failed` status stops dobby: held. `dobbyengine/passes.go:97` plans passes only for `running`.
8. D2, the Land way forward `lym fabric add <slug> --adopt-local`: **broken**. It hits the same answer that made D1 permanent.
9. D3 rationale, the reached-parent test "returns false unless batten stands at Land or Worktree-Teardown": **broken**. At Land with the child gone it returns false.
10. D3, appending `lym batten cancel` to Finalize's stop: **broken**. The cancel lifts the slug's own pauses, but that stop says "leaving the owner pause standing".
11. D4 Constraint, "D4's push is a teardown's push", read as outside `PATTERN-push-both-sides`: **broken**. That pattern covers every Go push of run records.
12. D4, the tag push needs no remote-refs lock and the branch push runs under it: held (`PATTERN-remote-refs-lock`).
13. D4, a covering pause holds `Remove` where it holds it today: held. The tag push already passes the gate while the teardown lock is held.
14. D4, archiving a diverged origin tip leaves nothing `Add` reads as live: held for `Remove` (`remoteleftover.go:117-125`).
15. D6, the lease is "the tip D4 archived", on every deletion path: **broken** for the cleanup sweep.
16. D6 bound, a kept branch "is another machine's live branch": weak but harmless. Processes on this machine push without the teardown lock, but they push the already-archived local tip.
17. D7 rationale, no new pair's probes interleave with the sweep: partly broken. The sweep takes the lock only before its remote deletes, and its archive and local delete run outside it. The checked-out-branch gate limits the damage.
18. D7, the wait runs out before any mutation: held. The probes come before the first mutation (`add.go:377-404`).
19. Constraint, the cancel's spawned push is the prime's records push: held (`cancelcmd.go:330-333`, `spawnStatusPush(location, "")`).
20. Constraint on archive-tag cover: held (`remoteleftover.go:100-126`).

**Findings**

1. **BLOCKING · consistency · D4 step 2 and Constraints ("`PATTERN-push-both-sides` … D4's push is a teardown's push")**
   - D4 pushes the weft branch alone, without the code branch.
   - A failed push leaves no failed-push record and no notice.
   - `PATTERN-push-both-sides` binds "Pushing run records from Go": the code branch goes along, and a push refused after retry leaves a failed-push record and a creel notice.
   - The weft branch carries run records, so the pattern applies. The record neither declares a change to it nor cites the departure from its text.
   - Evidence: `pattern/PATTERN-push-both-sides.md` (whole); `decision-record.md:74,116`.

2. **BLOCKING · design · D1 step 3**
   - D1 passes "the run's parent and anchor" to `Add`.
   - D1 only re-adopts live pairs, and `add.go:398` refuses any non-empty `opts.Anchor` when `weftOld.live`. The refusal does not check whether the anchor equals the recorded one.
   - Counterexample: on a subpath-anchored hub, every recoverable resume gets that refusal. D2 reads it as permanent and fails the run, which defeats D1.
   - Evidence: `internal/fabricengine/add.go:397-401`.

3. **MEDIUM · design · D1 bound**
   - Counterexample: `Remove` archives the tip, deletes the weft worktree, then fails to delete the local weft branch.
   - `removeSibling` tolerates that failure because the weft worktree is gone, so `Remove` succeeds and the covered branch stays local.
   - `probeWeftLeftover` with `localExists` returns live and never reads the archive tags.
   - D1 then re-creates a pair a removal tore down on purpose, against its stated bound.
   - Evidence: `remove.go:520-532`; `remoteleftover.go:88-93`; `add.go:178`.

4. **MEDIUM · design · D2, permanent vs transient answers**
   - "Neither a transport or probe failure nor the teardown-lock wait" cannot be implemented as written. `Add`'s probe errors are plain `fmt.Errorf` wraps of `gitexec` errors, with no sentinel (`remoteleftover.go:84-87,114-116`; `add.go:415-417`).
   - Operational refusals would be read as permanent and fail the run for good. Examples:
     - `ErrSourceDirty` when `Add` runs from the prime with `Parent` empty (`add.go:240-246`);
     - a `git worktree add` failure on a lock file (`add.go:431-432`);
     - the anchor-move stop (`add.go:260-266`).
   - The record does not say how each refusal class is told apart.

5. **MEDIUM · design · D2, Land way forward**
   - For a not-live sentinel, a hand `lym fabric add <slug> --adopt-local` is not adopt-only.
   - It forks a fresh weft branch over the archived leftover, so the child's records are lost. If the local warp branch is gone, `probeWarpLeftover` refuses with `ErrRemoteLeftover`.
   - For a permanent refusal, such as a records worktree directory still standing, the hand add refuses exactly as D1's `Add` did.
   - The named way forward does not recover the case it is named for.
   - Evidence: `remoteleftover.go:96-106,162-176`; `add.go:329-331`.

6. **MEDIUM · consistency · D3 Rationale, and D3's warning in `lym fabric remove`**
   - `childReachedParent` returns true only at Worktree-Teardown or past the merge (`cancel.go:691-697`).
   - At Land with the child status gone it returns false at `cancel.go:698-700`, so the "Land" half of the rationale is false.
   - D3's warning names `lym batten cancel` for every record not `done`, Land included.
   - An operator who follows it at Land releases a landed entry, the result D2 says to avoid.
   - Evidence: `internal/battencli/cancel.go:691-708`.

7. **MEDIUM · design · D3, the Finalize stop**
   - Finalize's stop says to remove the pair "leaving the owner pause standing" (`finalize_owner.go:160`).
   - D3 appends `lym batten cancel <slug>`, and the cancel's `clearRunPhase` lifts the slug's own pause entries (`cancel.go:469`; `cancelcmd.go:475-495`).
   - The owner pause is an entry of the slug's own scope with `OwnerCaller` (`pauserecord/record.go:208`).
   - D3 would make the stop contradict itself, and the record does not address it. I did not read `slugPauseRefs`'s filter.

8. **MEDIUM · design · D6 Decision, for the cleanup sweep**
   - D6 says the lease is "the tip D4 archived", but D4 changes only `Remove`.
   - `Topology.Cleanup` archives through `archiveWeftTip`, which tags the local tip only and never reads origin (`archive.go:84`, `resolveArchiveTip` prefers the local branch; `cleanup.go:245`).
   - Counterexample: an orphan weft branch whose origin tip is ahead of or diverges from the local tip. Leased on the origin tip it reads, the sweep deletes an origin tip no archive tag covers, so those records are lost from origin.
   - Evidence: `cleanup.go:243-259`; `archive.go:138-147`.

9. **LOW · design · D7, the cleanup sweep**
   - The sweep takes the lock "before that slug's remote deletes", after its archive tag push and its local `branch -D` (`cleanup.go:245-258`).
   - A concurrent `Add` can read the local weft branch as live (`add.go:333,384`) while the sweep deletes it.
   - `Add` then fails at worktree creation, or the gate refuses the sweep's delete.
   - The rationale "no new pair's origin probes … interleave with the … sweep" does not hold for the local half.

Tool calls:
- Grepped dobby's run-state checks, the owner-pause sites in `landingshed`, and the cancel's `LiftPauses`/`SpawnPush` seams. The manifest lacked dobby's pass planning and the cancel seams' bodies.
- Read `liftSlugPauses` and grepped `pauserecord` for the owner/hold callers. I needed to know whether a cancel lifts the owner pause, which the manifest did not cover.
