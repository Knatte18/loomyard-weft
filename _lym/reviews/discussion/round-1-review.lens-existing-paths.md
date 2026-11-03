```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code paths that already run against each decision's changes.

**Paths traced, per decision**

- **D1**, `originTargets` stops erroring on an absent pair. Six callers reach it:
  - `battencli` resume (`resumereconcile.go:36`): survives.
  - `fabriccli` remove: survives, because its gone-path skips the reconcile (`remove.go:57`).
  - `battenshed` OwnerCheck (`wire.go:1065`), loom `ownercheck.go:58` and `landingdeps.go:653`: these now get a verdict where they used to get an error. The record says nothing about them.
  - `fabricengine.DropLocal`, reached from `lym adopt` (`destroy.go:1898`, `adoptcli/adopt.go:740`): breaks.
- **D1**, a continue pass with the worktree absent is refused (`arm.go:621`). Dobby reads the refusal as a full pass owed, so it survives.
- **D1**, `AdoptLocal` set by an unattended run: the invariant written in `add.go:50`, `fabricengine/doc.go:702` and `fabriccli/doc.go:35` is contradicted. The other forks own that question.
- **D2**, a `failed` status: dobby's `planPass` plans no pass for a state that is not running (`dobbyengine/passes.go:97`). Survives.
- **D2**, `WriteFailedStop` called from the pre-run: the pre-run only probes the run lock and does not hold it (`arm.go:899`). Survives.
- **D3**, `lym batten cancel` added after Finalize's "leaving the owner pause standing": breaks. Details below.
- **D3**, warning in `lym fabric remove`: `fabriccli` already reads batten's shed status at the prime (`anchordeps.go:165`), so the read has a precedent. Survives.
- **D4**, weft branch push inside `Remove`: Remove still clears both failed-push records at its end (`remove.go:356`), which matches "no failed-push record". Survives.
- **D6**, lease in `removeSibling`'s origin-only path: the archive there resolves the origin tip (`archive.go:149`), so the lease tip is archived. Survives.
- **D6**, lease in the cleanup sweep (`Topology.Cleanup`): breaks. Details below.
- **D7**, detached pushes during Add's exclusive hold: the new pair's push is spawned after Add returns, as before (`wire.go:989`, `spawn.go:221`). Survives.
- **D7**, `Remove`'s respawn after a failure runs once the lock is released (`teardownlock.go:142`). Survives.

**Findings**

1. **MEDIUM · design · D1.** The run-scope reconcile change has no bound on which callers it reaches.
   - D1 says "The run-scope origin reconcile no longer errors when the run's pair worktree is absent". The error is raised in the shared `originTargets` (`internal/fabricengine/originreconcile.go:633`).
   - `fabricengine.DropLocal` calls `originTargets(l, ReconcileScope{Run: slug})` (`internal/fabricengine/destroy.go:1898`), reached from `lym adopt` (`internal/adoptcli/adopt.go:740`). With the error gone, an adopt over an absent pair gets no targets and moves nothing, silently, where today it refuses.
   - Loom's owner check (`internal/loomcli/ownercheck.go:58`), loom's landing reconcile (`internal/loomcli/landingdeps.go:653`) and batten's child OwnerCheck (`internal/battencli/wire.go:1065`) also change: a missing worktree, which used to be an error, now yields a normal verdict.
   - The record does not say whether the tolerance belongs only to batten's resume, for example as an option on `ReconcileScope`, or to every run-scope caller.
2. **MEDIUM · design · D3.** The cancel that D3 adds to Finalize's way forward either lifts the owner pause that the same stop says to leave standing, or refuses.
   - The stop text ends "then run `lym fabric remove <slug>`, leaving the owner pause standing" (`internal/landingshed/finalize_owner.go:160`).
   - The owner pause is a slug-scope entry: `tenderengine.Pause` with `Slug: slug`, `Caller: OwnerCaller` (`internal/tendercli/deps.go:222`).
   - Cancel's `liftSlugPauses` lifts every live entry whose scope is the slug, whatever its caller (`internal/battencli/cancelcmd.go:463-494`). Run as D3 says, the cancel therefore lifts that pause.
   - When the slug's owner is another machine, cancel's absent-pair path refuses instead with `cancelOwnedElsewhere` (`internal/battencli/cancel.go:512-518`). The appended step then cannot run.
   - D3 says nothing about either case.
3. **MEDIUM · design · D6 + D7 (cleanup sweep).** In the sweep, the lease can delete an origin tip that no archive tag covers.
   - D6 leases the sweep's weft delete on "the origin tip the deleter read under the slug's teardown lock, the tip D4 archived". D4 changes only `Topology.Remove`.
   - `Topology.Cleanup` archives through `archiveWeftTip`, which resolves the local tip whenever a local branch exists (`internal/fabricengine/archive.go:141-146`). It never reads or archives the origin tip (`internal/fabricengine/cleanup.go:245-258`).
   - So a sweep whose origin weft tip has diverged from the local tip would read that tip, lease on it and delete it. The weft records on that tip would then exist only as unreachable objects, with no archive tag. This is the condition D4's second tag exists to prevent.
   - The record does not extend D4's two-tip archive to the sweep.

**Tool calls**

1. Grep for `ReconcileScope{Run` callers, dobby failed handling and cancel's `LiftPauses`: my context had only the batten caller of the run-scope reconcile.
2. Read `dobbyengine/passes.go` runLabel, the `DropLocal` head, wire.go's OwnerCheck and `liftSlugPauses`: these are the bodies behind the call sites the first call found.
3. Grep for `planPass`, `slugPauseRefs`, `PauseOwnedElsewhere` and `DropLocal` callers: to confirm dobby skips non-running runs, the owner pause's scope, and where `DropLocal` is reached from.
4. Grep for run-lock acquisition around the pre-run and for `WriteFailedStop` callers: to check whether D2's write would report Busy against the pass's own lock.
