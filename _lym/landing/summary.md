# Adopt keeps the new owner idle; the old owner archives tips and exits

## What changes

During `lym adopt` on the adopting machine, the reconcile pulls the foreign owner record before this machine's owner write.
The orch watcher then stopped the orch, and the dobby could start a continue pass mid-takeover.
On the old owner, an idle machine never learned it had lost the subject, and its unpushed commits stayed where the new owner could not see them.

**Adopt in flight (D4).**
`adoptrecord.Record.Covers` reports whether an unfinished record lists a subject whose owner write has not stood:
`Inputs.PrimeRecorded` for `root`, `Inputs.OwnerStood` for a beam.
`adoptrecord.Watch` is the read-only accessor built on it.
It resolves a worktree to the subject whose owner it reads, walking up through the parent function to `root`,
so a beam with no owner of its own is covered by an adopt of `root`.
It reads the record before the board and takes no lock.
A record or board that does not read counts as no adopt, logged once per episode.
`orchcli.UnderAdoptWatch` builds it over the local board and `hubgeom.ParentOf`, and hands it to both workers:

- The orch watcher's owner check (`orchengine.OwnerWatch.UnderAdopt`) leaves the strand up while the adopt covers the beam.
- The dobby (`dobbyengine.Deps.UnderAdopt`) sets an adopt hold on the probe tick.
  Under the hold it spawns no pass and starts no task.
  It also runs no tidy, test-map or landing step, and does none of the hand-over below.
  This works the same way as the existing anchor-move hold.

Each hold's start and end log one line.

**Old owner hands over (D11).**
On a probe tick that reads the beam owned elsewhere with no adopt covering it, the dobby now:

1. owner-pauses each run it drives that is neither done nor already recorded owned elsewhere, through the existing `ownedElsewhere` path;
2. archives the local tips origin lacks through the new `Deps.ArchiveOwner`;
3. exits as before, once the archive succeeds.

A failed archive is logged once per episode and keeps the dobby up, retrying at each probe tick.
A run whose owner pause stands gets no pass meanwhile.
A run whose pause failed gets its full pass each probe tick, as it already did.

The probe now pulls Bolt through `Deps.PullBolt`, at most once per `boltPullInterval` (five minutes), before it reads the owner.
That way an idle machine learns a new owner, and the same probe acts on it.

**The archive.**
`fabricengine.ArchiveOwnerTips` takes `OwnerArchivePair`s, each a slug and a code worktree.
Under each slug's teardown lock, taken exclusive with the status-push wait, it tags each tip origin lacks.
The tips are the branches checked out in the code and records worktrees.
A tip is tagged `archive-owner/<slug>/<sha12>` (`OwnerArchiveTagName`) and the tag is pushed to that side's origin, with no branch push.
Origin lacks a tip when its copy of the branch is absent, or is neither the tip nor a descendant of it.
`createArchiveTag` and `tagAndPushArchiveTip` were split so that the existing archive and the new one share `createTagAt` and `pushArchiveTag`.
`beamcli.archiveOwnerTips` gathers the pairs: the worktree's own pair (`root` for the prime) and every run `battencli.DobbyRuns` lists under it.
The dobby's deps and `lym beam start` both call it.

**`lym beam start`.**
`orchcli.BeamUpOptions.ArchiveOwned` runs before the owner refusal stands, and its error joins the refusal with "a re-run of `lym beam start` retries it".
`lym beam start` sets it to the archive, skipped while an adopt in flight covers the worktree.
The prepare step's owner pauses run before it, as they already did.

## Design decisions

- `archive-owner/` is a sibling prefix, not a sub-path of `archive/`.
  `ArchiveTagSlug` and `ArchiveTagPrefix` match neither the prefix nor its tags.
  So the run, cost, timing and raddle reports, `TagRefs("archive/")` and the remote-warp cleanup never see these tags, with no change to any reader.
- The accessor is asked only once the owner reads foreign, in both workers.
  The record is still read first inside the accessor, so a hub with no unfinished record never reads the board for it.
- The accessor's constructor sits in `orchcli`, not in `adoptrecord`.
  Building it over `hubgeom.ParentOf` inside `adoptrecord` would import `hubgeom`, which pulls in most of the engines.
  `adoptrecord` instead takes the owners read and the parent function as seams.
- The adopt hold is set only on probe ticks, where the owner is read, and holds until a later probe clears it.
- The archive takes each pair's teardown lock exclusive, as the teardown's archive does, waiting at most `StatusPushLockWait`.
- A detached HEAD in a pair worktree fails that pair's archive instead of skipping it, so the tip is never silently left out.

## To check by hand

- `beamcli.archiveOwnerTips` and `ownerArchiveBeforeRefusal` have no test of their own.
  Tests pin the orch side of the refusal (`TestStart_ArchivesBeforeRefusingAForeignOwner`), the dobby side (`TestRun_ForeignOwner`) and the fabric call (`TestArchiveOwnerTips`).
  Check the pair gathering by hand, especially a beam dobby whose runs live under the prime's status files.
- `TestArchiveOwnerTips` pushes the hubforge fixture's records tip before each row, since the fixture leaves it unpushed.
- The `watch` verb builds its under-adopt accessor the first time a foreign owner makes the watcher ask, and keeps it.
  A watcher that cannot resolve the hub's prime counts as under no adopt, logged once per episode.
- A dobby that cannot list its runs at hand-over still archives and exits; only a Warn records that no run was paused.
  Confirm that is acceptable, or whether the hand-over should wait for a run listing.
- An abandoned adopt record keeps this machine's orch and dobby of the covered worktrees idle under the foreign owner until `lym adopt --discard` or a resume, as D4's bound states.
  `lym beam status` shows the record.
