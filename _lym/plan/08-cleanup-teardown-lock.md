# Card 8 — cleanup-teardown-lock

**Edit:**
- `internal/fabricengine#Topology.Cleanup`
- `internal/fabricengine#CleanupBranchEntry`
- `internal/fabricengine#Topology.cleanupRemoteWarp`
- `internal/fabricengine_test#TestCleanup_ApplyArchivesEachOrphanBeforeDeleting`
- `internal/fabricengine_test#TestCleanupRemoteWarp_ApplyDeletesOnlyTheCandidate`
- `PATTERN.md`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#Topology.lockPairTeardown`
- `internal/fabricengine#TeardownLockPath`
- `internal/fabricengine#ErrPairBusy`
- `internal/fabricengine#RemoteWarpBranchEntry`

**Intent:** Decision record D7's third bullet and D6's "under the slug's teardown lock": the cleanup sweep takes each slug's teardown lock with one try before it archives or deletes anything of that slug.

- In `Topology.Cleanup`, under `apply` and for an orphan that is not local-only, the sweep calls `lockPairTeardown` with `try` true (and the card-5 `rerun` text `re-run the cleanup once it finishes`, which names no verb since more than one command reaches the sweep) for the orphan's slug before `archiveWeftTip`, and releases it after the orphan's remote delete.
  A held lock (`errors.Is(err, ErrPairBusy)`) skips the slug whole: no archive, no local delete, no remote delete; the entry gets `Skipped bool` (`skipped`, omitempty) and `SkippedReason` naming the held teardown lock and the re-run, and the sweep goes on with the next branch without an error.
  Any other lock error fills the entry's `Error`, as an archive failure does, and keeps the branch.
- So the origin tip card 6's archive reads and card 7's lease uses is read under the lock.
- The remote warp sweep (`Topology.cleanupRemoteWarp`, `lym fabric cleanup`'s deletion of former pairs' task branches on origin) takes the same one try per candidate's slug before its leased delete; a held lock keeps the candidate with `Deleted` false and a `Reason` naming the held teardown lock and the re-run, so no new pair's `Add` interleaves with a deletion of the slug's former task branch (the Goal's "no removal or deletion of a slug's former pair touches a later run's branches").
- Bound: a skipped slug is swept by a later run; the try never waits, so the sweep is never held up by a pair's push or removal.

PATTERN entry, prescribed verbatim, added to `PATTERN.md`'s `## Fabric and git` section after `PATTERN-push-both-sides`:

```markdown
- `PATTERN-slug-remote-order` — Touching a slug's pair or its origin branches: under fabricengine's per-slug teardown lock, shared for each pair push outside a removal, exclusive for Add, Remove, task-branch discard and cleanup sweep; a weft branch delete on origin is leased on the origin tip read.
```

Extend `internal/fabricengine/doc.go`'s cleanup paragraph with the per-slug try and the skip.

Tests:
- `TestCleanup_ApplyArchivesEachOrphanBeforeDeleting` gains a row whose test holds the orphan's teardown lock (`lock.AcquireWriteLock(TeardownLockPath(...))`) during `Cleanup(apply, remote)`: the entry reads `Skipped` with its reason, no archive tag of the slug is on origin, and the branch stands locally and on origin; a second orphan in the same sweep is archived and deleted.
- The same test gains a row where a file stands at the teardown lock's directory path (`<HubScratchDir>/teardown`), so `ensureTeardownLockDir` fails: the orphan's entry carries `Error`, the branch stands locally and on origin, and the sweep returns no error.
- `TestCleanupRemoteWarp_ApplyDeletesOnlyTheCandidate` gains a row holding the candidate's teardown lock: the candidate is kept with the lock reason and its origin branch stands.

**ImpactSummary:** One more step in each of the two sweeps' per-branch loops and two omitempty fields on `CleanupBranchEntry`; `lym fabric cleanup` and the hub reconcile's sweep keep their signatures, and the PATTERN overview grows by one entry line, well under the headroom line.

**Commit:** `8: The cleanup sweeps skip a slug whose teardown lock is held`
