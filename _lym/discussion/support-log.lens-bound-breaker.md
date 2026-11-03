```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I tried to break the record's claims and bounds with concrete counterexamples.

**Findings**

- **BLOCKING, D2 with D1 and D7: a transient failure becomes a failed run whose way forward is a cancel that loses work.**
  - D2 fails the run "when `Add` refuses", and sends every refusal to `lym batten cancel <slug>`.
  - `Add` probes origin before its first mutation (`probeWeftLeftover`, `probeWarpLeftover`). An unreachable origin makes those probes error.
  - D1 runs ahead of the reconcile, and the reconcile is the step that handles an unreachable origin gently (Warn, origin-owed marker, resume).
  - D7 adds a second transient refusal: the wait for the teardown lock runs out while a push of the slug holds it.
  - Counterexample: an offline machine, or one push still holding the lock, on a run with a live weft branch on origin. The pass writes `failed` and names cancel. Cancel clears the record and the claim, although the next online pass would have re-adopted the pair.
  - Needed: classify a probe or transport failure, and `ErrPairPushRunning`, as transient. Retry, or fall back to today's refusal, instead of writing `failed`. See `PATTERN-transient-stop`.

- **MEDIUM, D1: `Add` "as the Worktree-Create row does" refuses the usual worktree-gone states, so D1 rarely re-adopts.**
  - `internal/fabricengine/add.go` refuses an existing unmarked local warp branch with `ErrBranchExists` unless `AdoptLocal` is set. That check runs before the liveness probe, so it fires for a live pair too.
  - It also refuses a records worktree directory that already exists ("weft worktree directory already exists").
  - Counterexample 1: a task worktree directory deleted by hand, or left as a stale git registration, with both local branches kept. The result is `ErrBranchExists` and then D2's failed stop.
  - Counterexample 2: a partial removal ("warp worktree removed, but weft teardown failed and the weft worktree remains"). The weft-directory refusal fires, then D2.
  - D1 re-adopts only when both local branches are gone, which in practice means a pair whose branches live only on origin. The record says nothing about `AdoptLocal`, the worktree prune, or a surviving records worktree.

- **MEDIUM, D2 and D3: the cancel way forward can undo a landed run.**
  - D1 and D2 apply at every row but Worktree-Create and Worktree-Teardown, so Land is included.
  - With the pair gone, `childReachedParent` returns false at Land. The child is not found, and that check comes before the `pastRunShed` test (`cancel.go`).
  - Counterexample: a run at Land whose worktree is gone and whose weft tip is archived. D2 names `lym batten cancel`, and cancel clears the claim to draft. The work has already merged, so the entry is re-released instead of marked done.
  - The record's claim that with the pair gone cancel "runs no reached-parent refusal" is literally true, but that is exactly the hazard. D2 needs a different way forward for a run past Run-Shed.

- **MEDIUM, D1 bound: an owner check on a stale local board lets a foreign pair be re-created.**
  - D1 checks only the local board before `Add`. `Add` then fetches and adopts the live pair's branches from origin and spawns its push.
  - Only afterwards does the reconcile judge origin as owned elsewhere and refuse.
  - Counterexample: after `lym adopt` on another machine, this machine's local board is stale. The pass re-creates the other machine's live pair here, and it stays on disk with the refusal repeating each pass.
  - The record's purpose of "never one owned by another machine" breaks. Fix: run a judge-only origin read, or the reconcile, for the owner verdict before `Add`.

- **MEDIUM, D6 bound: a branch kept by the lease can be adopted as live, not treated as a leftover.**
  - D6 says a kept branch is either the later run's, or one "`Add` treats as a leftover by its existing rule".
  - `probeWeftLeftover` treats an origin tip as replaceable only when an archive tag's target equals or descends from it. Any other tip makes the pair live, and `Add` adopts it.
  - Counterexample: origin's weft branch has diverged, D4's push is refused as not fast-forward, and the archive tag sits at the local tip. D6's lease refuses, so origin keeps a tip that no tag covers. The slug's next run adopts the cancelled run's diverged records.
  - Also, when D4's push fails on transport, the local tip is ahead of origin. D6 then reports "kept because origin moved", but origin did not move.

- **MEDIUM, PATTERN changes and D7 against the Constraints: `Topology.Cleanup` is outside the lock and the claim.**
  - `internal/fabricengine/cleanup.go` deletes weft branches on origin (`deleteWeftBranchOnRemote`) and never takes `lockPairTeardown`. D7 does not extend it.
  - The proposed entry `PATTERN-slug-remote-order` says touching a slug's pair or origin branches happens only under the lock, so `Cleanup` would violate it as written.
  - The Constraints line "D6 and D7 order every deleter of a slug's branch against a later run whatever the trigger was" fails for `Cleanup`. Only the lease and the checked-out gate protect it, and `Add` can adopt the same local weft branch, so the lease tip is the same.

- **LOW, D4: the in-process coalesced push sends a failure notice for a branch about to be deleted.**
  - When the push is refused after its retry, `PATTERN-push-both-sides` makes it write a failed-push record and post a creel notice.
  - The branch is deleted moments later, so the orch gets a notice about a branch the removal is deleting. The record says nothing about this.

- **LOW, D4: "refused" is undefined beyond not-fast-forward.**
  - The record does not say whether an auth or transport failure of the weft push fails the removal. The Bound says only "a refused branch push".

**Claims attacked**
- D1: "only a live pair": held, since coverage is by ancestor, so an archived older tip on origin is not live. "Re-adopts like Worktree-Create": broken (`AdoptLocal`, surviving weft directory). Owner bound: broken (stale local board).
- D2: "status leaves running, so dobby stops": held. "Only when no live pair is left": broken (transient refusals). Way forward: broken at Land.
- D3: "cancel after fabric remove runs no reached-parent refusal": held, because `RecordsPresent` keeps cancel going. Hazard at Land as above.
- D4: "a detached push started under the held lock exits without pushing": held. The push lock and remote-refs lock under an exempt cancel: held (`SetGateExemptAll`). Failure modes: underspecified.
- D5: held.
- D6: "kept is the later run's or a leftover": broken (diverged tip is adopted as live). "Moved" wording: broken when local is ahead.
- D7: "Add releases before the caller spawns the push": held. No self-deadlock with `DiscardTaskBranch`, whose only caller is cancel after `RemovePair` returns: held. "Every deleter is ordered": broken (`Cleanup`).
- Constraints: "cancel deletes synchronously and its push deletes nothing": held. Task branch delete in Remove is leased: held, via `deleteTaskBranchAtTip`.

**Tool calls**
1. `internal/fabricengine/remoteleftover.go`, `probeWeftLeftover` and `probeLiveLocalWeft`: to learn what archive coverage means and what makes a pair live. My context had only the file head.
2. `internal/battencli/cancel.go` `refuseNothingLeft`, `cancelcmd.go` `ChildReachedParent`, and the callers of `DiscardTaskBranch`: to test the D3 claim and the D7 deadlock. My context lacked those spans.
3. A grep of `internal/fabricengine/add.go` for its refusals (`AdoptLocal`, `ErrBranchExists`, existing worktree directories): D1 depends on them and my context had not read `Add`.
4. A grep of `internal/pauserecord` for gate exemption: to check whether D4's locked push could block a cancel on a pause. My context lacked it.
5. `internal/fabricengine/remove.go`, `deleteTaskBranchOnOrigin`, and the lock use in `cleanup.go`: to check whether the task-branch delete is already leased and whether `Cleanup` takes the teardown lock. My context lacked both.
