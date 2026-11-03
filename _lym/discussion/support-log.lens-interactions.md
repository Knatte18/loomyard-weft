```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of the record's decisions that act on the same state, and the acceptance criteria and constraints against them.

**Findings**

1. **MEDIUM — D1 × D2 × D7: a transient refusal from `Add` becomes a permanent failed run.**
   - D2 fails the run whenever `Add` refuses during re-adoption.
   - D7 makes `Add` refuse when its teardown-lock wait runs out while a former push or removal of the slug is still running.
   - `Add`'s origin probes (`remoteHeadTip`, `remoteArchiveTags`, the FETCH_HEAD fetch in `probeWeftLeftover`) also return errors when origin cannot be reached.
   - Today `reconcileResume` resumes with an origin-owed marker when origin is unreachable. With D1 → D2 in force, the same condition writes `failed` with the way forward `lym batten cancel`, and that cancel throws the run's record away.
   - D2's bound ("only when … no live pair is left") does not exclude these cases.
   - Fix: D2 fails the run only on a non-live pair or a permanent `Add` refusal. A lock timeout or a network error refuses the pass as today, so dobby retries.

2. **MEDIUM — D1 has no adopt-only form of `Add`, so a non-live slug gets a brand-new pair.**
   - D1 says "Only a live pair is re-adopted, by `Add`'s own liveness rule", and D2 fires "when D1 finds no live pair".
   - `Topology.Add` (`internal/fabricengine/add.go`) adopts a live pair but forks a fresh one otherwise. It has no mode that refuses on a non-live pair; the only variants are `Add`, `AddLocal` and `AdoptLocal`.
   - Calling `Add` as D1 describes would therefore re-create an empty pair from the parent after `lym fabric remove`, and the run would resume on it. That breaks D2's case (archive-covered tip → failed) and D1's own bound.
   - The record must decide how to get adopt-only behaviour: a liveness probe before `Add`, or an `AddOptions` flag that refuses a fork.

3. **MEDIUM — D4 × D6 × D1: a weft branch kept on origin reads as live and D1 brings it back.**
   - D4: when the weft branch push is refused as not fast-forward, the tag goes out alone at the local tip.
   - D6: the leased delete then fails because origin is not at the local tip, so the branch is "kept because origin moved".
   - `probeWeftLeftover` counts an origin tip as covered only when the tag's target equals it or descends from it. A diverged origin tip is not covered, so it reads as live.
   - Resulting path: after `lym fabric remove --remote` or `lym batten cancel --remove`, the next D1 pass on a still-standing record re-adopts that kept branch. A later fresh run's `Add` also adopts it instead of forking.
   - This contradicts D6's bound ("one `Add` treats as a leftover by its existing rule"): by `Add`'s rule a branch with no covering tag is live, not a leftover.

4. **LOW — Constraints / D6 × D7: "D6 and D7 order every deleter" overclaims.**
   - D7 locks `Add`, `Remove` and `DiscardTaskBranch`.
   - The cleanup sweep's weft delete (`deleteWeftBranchOnRemote` reached from `Topology.Cleanup`) gets D6's lease but no lock, and the PATTERN entry text also omits Cleanup.
   - A cleanup that reads origin's tip and then deletes can still interleave with a new run's `Add`, its first push and the read of origin's tip. The lease only protects against a tip that moves after the read.

5. **LOW — D7 × `DetachedPush`: while `Add` holds the lock exclusive, a push is dropped and never re-sent.**
   - A push child of the same slug that starts while `Add` holds the lock exclusive (for example a concurrent hand pass during D1's re-adoption) fails `TryAcquireReadLock`.
   - It logs "the pair is being removed; its push is not run" and exits with nil, so its commits stay local.
   - Unlike a failed `Remove`, nothing respawns it, and the log text misreads `Add` as a removal.

**Pairs checked**

- D1 × D2 (`Add` refusal → failed stop): broken (findings 1 and 2).
- D1 × D7 (`Add` under the teardown lock, called from the pre-run): broken for a lock timeout (finding 1). The ordering otherwise holds: `Add` releases the lock before the pair push is spawned.
- D1 × D3 (`fabric remove` then a dobby pass): held only if `Add` can be told not to fork (finding 2).
- D1 × D4 (the archive tag decides liveness): held when the branch push succeeds or origin is behind the local tip; broken when it diverges (finding 3).
- D1 × D6 (a kept origin branch read as live): broken (finding 3).
- D1 × the owner-machine Worktree-Teardown exception: held. The rows are disjoint.
- D2 × D3 (`lym batten cancel` as the way forward with the pair gone): held. In `cancel.go` a standing run record counts as `otherLeft`, so the cancel goes on rather than refusing with nothing-left.
- D2 × dobby pass planning (`failed` stops passes): held.
- D4 × D6 (the leased delete after the branch push, lease = local tip): held on the normal path; broken on a not-fast-forward refusal (finding 3).
- D4 × D7 (`Remove`'s longer hold under the lock, `Add` waiting the same teardown wait): held. `Add` can time out more often, which feeds finding 1.
- D6 × D7 (cancel's `Remove` then `Discard` take the lock separately, and `Add` could slip between them): held, because cancel clears the claim only after the discard, so dobby cannot start a new run between them.
- D6 × cleanup sweep × D7: partly broken (finding 4).
- D7 × `DetachedPush` read-lock try: weak (finding 5).
- D5 × the rest: independent; held.

**Acceptance criteria and constraints**

- D2's criterion "a resume whose re-adoption `Add` refuses writes a `failed` status" turns finding 1's transient cases into failed runs.
- D1's first criterion holds only with an adopt-only `Add` (finding 2).
- D6's "kept" criterion does not consider D1 re-adopting the kept branch (finding 3).
- The Constraints line "D6 and D7 order every deleter" is broken by the cleanup sweep (finding 4).
- The other criteria hold against their decisions.

**Extra tool calls**

- Read `internal/fabricengine/remoteleftover.go` lines 60–130, to learn what "covers" means for archive-tag liveness; my context had only the file head.
- Read `internal/battencli/cancel.go` lines 585–626 and grepped `add.go`/`cleanup.go`, to see whether cancel refuses with nothing-left when the pair is gone and whether `Add` has an adopt-only option; my context had neither.
