```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced each decision's state against the code paths that already write or read it.

**Paths traced, per decision**

- D1 → `originTargets` run scope without the option (`DropLocal`/adopt): survives.
- D1 → `ownerPreflight` at Worktree-Teardown and Worktree-Create (`owner.go:235-263`): survives, both rows are excluded.
- D1 → `refuseContinuePass` (`arm.go:623`): survives, the new refusal stacks on it.
- D1 → `Add` with no `Parent`: the fork-point marker check, the move-record stop and the prime HEAD read (`add.go:251-266`, `351-363`) still run under adopt-only. **Breaks** (finding 2).
- D1 → `Add`'s task-dir and records-dir existence checks (`add.go:298-331`): survives, D2 covers the records-dir check.
- D2 → `WriteFailedStop` (`failedstop.go:39-73`): survives. It does no commit, which D2 adds itself. Busy and NotRunning are handled.
- D2 → `checkRunStatus` resuming `failed` (`arm.go:586`): survives.
- D2 → the batten `CommitStatus` no-op skip marker (`commitstatus.go` header): survives, because the skip only declines identical commits.
- D2 → whether dobby skips passes for a `failed` status: unverified. Dobby's planner had no `StateFailed` read I could find, so I report no finding.
- D2 → the `ErrRemoteLeftover` way forward, then the re-adopted weft's pushes. **Breaks** (finding 1).
- D4 → the tag push's pause gate (`gitexec.go:226-238`, the network gate covers every remote subcommand): survives, the claim holds.
- D4 → `retryablePushRefusal` (`pushretry.go:67`): today it retries only `(failed)` and `cannot lock ref`. D4 adds a reason. Survives.
- D6 → the weft pushes that run without the teardown lock: **breaks** D6's bound (finding 3).
- D6 → `removeSibling`'s origin-only path and the `Cleanup` remote delete (both unleased today): survives as designed.
- D7 → the `Add` callers (`fabric.go:552`, `adoptcli/deps.go:75`, `battencli/wire.go:973`) and their push spawns after `Add` returns: survives (not traced past the call).
- D8 → the run-scope reconcile, which judges only branches that have a tracking ref: survives.
- D8 → the push's `checkOrigin`/`UpstreamSHA` on an adopted weft branch that keeps its upstream config: **breaks** (finding 1).

**Findings**

1. **BLOCKING, design.** D8 together with D1 (and D2's `ErrRemoteLeftover` way forward).
   - D8 deletes `refs/remotes/origin/<weft>` of an adopted local weft branch that origin lacks. The branch still has its `branch.<weft>.remote`/`merge` upstream config from the former pair.
   - `UpstreamSHA` finds the config, then fails to resolve the missing tracking ref. It returns a plain error, not `ErrNoUpstream` (`internal/gitrepo/blobread.go:281-289`).
   - `checkOrigin` returns `(false, upstreamErr)` on any error other than `ErrNoUpstream` (`internal/fabricengine/pushclassified.go:391-397`).
   - So every push of the re-adopted pair's weft is refused before it runs. That includes D1 step 4's pair push and the status pushes after it, so the run's records never reach origin.
   - Before D8 the same branch read gone instead (`pushclassified.go:409-410`, `ErrBranchGone`).
   - Cases hit: D1's "removal left the local weft branch" case and D8's second acceptance criterion. D2's hand `git push --delete`, which also drops the tracking ref and keeps the config, ends in the same state.
   - The record never mentions the upstream config.
2. **MEDIUM, design.** D1 claims "no parent is resolved" under adopt-only. `Add` with an empty `Parent` still checks the prime's weft marker (`refuseEmptyOwnMarker`), its move-record stop, and reads the prime's HEAD.
   - A detached or unborn prime HEAD refuses with "cannot spawn weft branch" (`add.go:351-361`).
   - That refusal is outside D2's closed set, so it is transient. Dobby then retries every pass with no end, which is the failure D1 exists to end.
   - None of these checks guards a fork the adopt-only option makes, and D1 names none of them.
3. **MEDIUM, design.** D6's bound says a branch the lease keeps "was pushed by another process … no new pair of this slug on this machine; it is another machine's live branch".
   - The teardown lock is held shared only by a push spawned with `TeardownSlug` set (`teardownlock.go:75-87`; `weft_verbs.go:266-271` passes it only in the detached child).
   - The synchronous `commit-records` push, which `PATTERN-push-both-sides` allows, takes no teardown lock. So does any spawn with an empty `TeardownSlug`. Either can land this same pair's weft tip between D4's read and the leased delete.
   - The lease then keeps a tip no archive tag covers. `probeWeftLeftover` reads it as live (`remoteleftover.go:105-106`, `126`), so a later `Add` re-adopts the removed pair.
   - D6/D7 do not say which weft pushes must hold the lock, so the bound rests on a premise the code does not guarantee.

**Tool calls**

1. grep for `checkOrigin` in fabricengine: no hits; my context lacked the push's origin check.
2. grep for `checkOrigin`, `TeardownSlug:` spawns and `Add` callers: the push origin check, the teardown-slug carriers and the `Add` callers were not in the manifest.
3. Read `pushclassified.go:380-440`: the `checkOrigin` body, needed for D8's effect on pushes.
4. Read `UpstreamSHA` (`blobread.go:264-292`) and `weft_verbs.go:255-285`: how a missing tracking ref with upstream config is read, and which push carries `TeardownSlug`.
5. grep for `StateFailed` in dobby: dobby's handling of a `failed` status (no hits).
6. grep for `"failed"`/`StateFailed` across dobby files: the same question, broader (only `start.go`, no planner read found).
