# Keep adopter's orch and dobby idle; old owner archives tips and exits

During an adopt, the adopting machine's orch watcher and dobby no longer react to the foreign owner they read.
Before this change the watcher stopped its orch when the reconcile pulled the foreign owner record,
and the local dobby could start a continue pass during the takeover.
The old owner, for its part, now pauses its runs and archives its unpushed tips before it exits,
so the adopting machine can reach commits it cannot otherwise see.

## What changes

- **Under-adopt accessor** (`internal/adoptrecord`): `Watch.Covers(slug)` is a read-only judgment of whether an adopt in flight on this machine covers a worktree.
  The record must be unfinished, list the worktree's subject, and record no owner write of it that stood.
  The local board must read the subject owned by another machine.
  A beam with no owner of its own is covered through `hubgeom.ParentOf`, up to an adopted `root`.
  It stats and reads the record without a lock; a record or board that does not read counts as no adopt, logged once per episode.
- **Orch watcher** (`internal/orchengine`, `internal/orchcli`): the owner check reads the accessor first,
  and while the worktree is covered the watcher neither stops its orch strand nor exits on the foreign owner.
  `orchcli` wires the accessor in, so the engine does not import `adoptcli`.
- **Dobby** (`internal/dobbyengine`, `internal/beamcli`): under an adopt the dobby stays up and starts no pass, continue passes included, and no task.
  Its probe also pulls Bolt through fabric at most once per `boltPullInterval` (five minutes),
  so an idle machine learns a new owner; a failed pull is logged once per episode.
  When the probe reads a foreign owner and no adopt covers the beam, the dobby hands the beam over:
  1. it owner-pauses each run it drives, so no pass of a run whose pause stands starts while it archives;
  2. it archives the local tips origin lacks;
  3. it exits.
  A failed archive is logged once per episode and keeps the dobby up; the next probe retries it.
- **Owner archive** (`internal/fabricengine`): `OwnerArchiveTagName` names `archive-owner/<slug>/<sha12>` beside `ArchiveTagName`,
  and `ArchiveOwnerTips` tags and pushes each local code and weft tip of the given pairs that origin lacks.
  Each tag goes to its own repository's origin with no branch push, under the slug's teardown lock.
  A re-run reuses the same tags and pushes what is missing.
- **`lym beam start`** (`internal/beamcli`): when no dobby runs on the machine, a subject another machine owns that no adopt covers
  is archived through the same fabric call before the refusal; a failed archive is named in the refusal, and a re-run retries it.

## Design decisions

- The `archive-owner/` prefix is separate from `archive/`, so no reader of `ArchiveTagSlug` or `ArchiveTagPrefix` matches it.
  The run, cost, timing and raddle reports would otherwise list a live run, `root` or a beam as archived,
  and the remote-warp cleanup would count the tags as its own.
- Tags rather than archive branches, as the teardown and `--drop-local` archive do.
  An archive-tag push moves no branch of the pair, so `PATTERN-owner-machine`'s rule on pushing does not cover it.
- The old owner pauses its runs before it archives, the reverse of the board's order, so no pass starts during the archive.
- The record is machine-local: the hold covers only this machine's watchers and dobbies, and ends at the subject's owner write.
  After that write stands, a later foreign owner is handled as before and by the hand-over, even with the record unfinished.
- Git runs only in `internal/fabricengine`; the Bolt pull and the archive are fabric calls.

## Known bounds

- An adopt abandoned before its owner write leaves this machine's orch and dobby of the covered worktrees up and idle until it is resumed or discarded.
- A commit made by hand in a worktree after the archive stays local.
- A dobby whose owner pause failed lets the run's full pass go on at each probe tick while the archive retries.
- A machine with no dobby running archives nothing until `lym beam start` runs there.

## To check by hand

Nothing was run by hand, and no regression check ran for this task. Check these findings, left open by the review:

- A beam with no owner recorded under an adopted `root` may not be covered by the under-adopt accessor in `internal/adoptrecord/watch.go`
  (severity BLOCKING); check that the watcher and dobby of such a beam stay up and idle during `lym adopt root`.
- The adopt hold in `internal/dobbyengine/handover.go` skips steps (pause, archive, pass starts) that no test pins; check that each is skipped under the hold.
- The decision rows for which tips the owner archive takes (origin absent, equal, ahead, diverged) in `internal/fabricengine/ownerarchive.go` are missing from its tests.
- The pairs the owner archive covers in `internal/beamcli/ownerarchive.go` and `internal/fabricengine/ownerarchive.go` are untested end to end.
