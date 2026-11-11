## Goal

The adopting machine's orch and dobby stay up and idle, and the old owner pauses, archives its unpushed tips and exits

Takes: adopt-worker-record's internal/adoptrecord and its record of the subjects whose owner write stood
Delivers: the under-adopt predicate over the record and hubgeom.ParentOf, handed as a read-only accessor to the orch watcher and the dobby, which keep up and idle under it; the dobby's periodic Bolt pull on its probe; the old owner's owner-pause, archive-owner tag push of each local tip origin lacks, and exit; lym beam start's archive before its foreign-owner refusal
Exports: the under-adopt accessor in internal/adoptrecord, the archive-owner tag name function beside fabricengine.ArchiveTagName, the fabric archive call

## Scope

The part changes these packages:

- internal/adoptrecord (about 100 changed lines)
- internal/orchengine (about 60 changed lines)
- internal/orchcli (about 160 changed lines)
- internal/dobbyengine (about 350 changed lines)
- internal/beamcli (about 150 changed lines)
- internal/fabricengine (about 350 changed lines)

## Decisions

### D4 — An adopt in flight keeps this machine's orch and dobby of its subjects up and idle

A subject (`root` or a beam) is under adopt when an adopt record on this machine is unfinished, lists the subject in its resolved subject set, does not list it among the subjects whose owner write stood, and the local board reads the subject owned by another machine.
A subject under adopt covers its own worktree and every beam worktree whose owner it reads through `hubgeom.ParentOf` from the subject: a beam with no owner recorded under an adopted `root`.
In each covered worktree, while the subject is under adopt:

- The orch watcher does not stop its orch strand or exit on the foreign owner; its owner check in `internal/orchengine/owner.go` reads the record first.
- The dobby does not exit on the foreign owner in `internal/dobbyengine/loop.go`, and runs none of D11's steps.
  It starts no pass, continue passes included, and no task under the subject.

Each reads the record through a read-only accessor its CLI wiring hands it, so neither engine imports `adoptcli`.
The accessor stats the record and reads it without a lock; a record that does not read counts as no record, logged once per episode.

Rationale: the orch watcher on the adopting machine stopped the orch when the reconcile pulled the foreign owner record, and the local dobby started a continue pass during the takeover.
The window that needs covering ends at the subject's owner write.

Bound: the record is machine-local and covers only this machine's watchers and dobbies of the covered worktrees, until the subject's owner write.
An adopt abandoned before its owner write leaves this machine's orch and dobby of those worktrees up and idle under the foreign owner until it is resumed or discarded;
`lym beam status` names it (D9).
Once the owner write of a subject stands, a later foreign owner is handled as today and by D11, even with the record unfinished.

### D11 — The old owner stops its runs and archives its unpushed tips

The old owner's local Bolt learns a new owner only on a Bolt pull: a board write's pull, a run's origin reconcile, `lym fabric pull` or a beam start.
An idle machine makes none, so the dobby pulls Bolt through fabric on a probe at most once per a named interval, logging a failed pull once per episode.

When a dobby's probe reads its subject owned by another machine, and the subject is not under adopt (D4), before it exits it:

1. Owner-pauses each run it drives, as a pass exiting owned elsewhere does today, so a run whose pause stands starts no pass while it archives.
2. Archives every local tip under the subject that origin lacks: the code and weft branches of the subject's own worktree and of each run whose parent is that worktree.
   A tip is archived under an `archive-owner/<slug>/<sha12>` tag, named by a new fabric name function beside `fabricengine.ArchiveTagName`, pushed to origin with fabric's archive-tag push and no branch push,
   under the slug's teardown lock as the teardown's archive is (`PATTERN-pair-remote`).
   A code tip's tag is pushed to the code repository's origin, and a weft tip's to the weft repository's origin.
   `root` is the slug of the prime's branches.
3. Exits, as today.

The archive is one fabric call, which the dobby's deps and `lym beam start` both wire.
A failed archive is logged once per episode and the dobby stays up, retrying at each probe; it exits once the archive succeeds.
Its runs stay owner-paused meanwhile, and batten's owner refusal stops a task start under the foreign owner.

A machine whose dobby is not running reaches the archive through `lym beam start`, which refuses a subject another machine owns before any dobby starts.
When the subject is not under adopt (D4), that refusal first runs the same fabric archive call for the subject, under the slug's teardown lock, then refuses as today.
Its prepare has already owner-paused each run the reconcile reads owned elsewhere, as today.
A failed archive adds its error to the refusal, and a re-run of `lym beam start` retries it.

Rationale: the adopting machine cannot see the old owner's unpushed local commits.
Tags, as the teardown and `--drop-local` archive, not new archive branches.
The `archive-owner/` prefix keeps these tags out of every reader of `archive/` tags, through `fabricengine.ArchiveTagSlug` or `fabricengine.ArchiveTagPrefix`:
the run, cost, timing and raddle reports would otherwise list a live run, `root` or a beam as an archived run, and the remote-warp cleanup would count them as its own.
`PATTERN-owner-machine`'s rule reads "Run start, resume, push or landing: only if no other machine owns its entry", and its push is a run's commit and push of its pair.
An archive-tag push moves no branch of the pair, so it is outside the rule;
the teardown's archive already pushes archive tags for a pair whose verdict is `owned_elsewhere`.

Bound: a commit made by hand in a worktree after the archive stays local.
A dobby whose owner pause failed lets the run's full pass go on at each probe tick as today, while the archive retries.
A machine whose dobby is not running archives nothing until `lym beam start` runs there.
`lym s` there runs `lym adopt --here` first, which takes the subject back, and that adopt's reconcile meets the tips through its local-commit stop.

## Constraints

- Each step is safe to run again: a re-run adds only missing pairs, pauses only slugs the record does not list as paused, writes the same owners, judges the deploy again, starts beam steps that leave a live pair alone, and starts no second live check.
- The adopt record and `adopt.lock` are never-tracked, under `fabricengine.AdoptScratchDir` (`PATTERN-lym-files`).
  The record is machine-local by nature: it covers only this machine's watcher and dobby.
- Git runs only in `internal/fabricengine` (`PATTERN-fabric-boundary`); the dobby's Bolt pull, the archive and the archive read are fabric calls.

## Auto-mode assumptions

- "No orch start when adopt was run from an orch session" means no orch start for any subject of that adopt.
- An unstarted released task of a subject is an adopted run for D5, since the dobby would otherwise start it.
- The old owner pauses its runs before it archives, the reverse of the board's order, so no pass starts while it archives.

## Open risks

## Acceptance criteria

- D4: with an unfinished record listing a subject that reads owned by another machine and has no owner write, the orch watcher keeps its strand and the dobby neither exits, archives, nor starts a pass or task under the subject.
- D4: during `lym adopt root`, the watcher and dobby of a beam with no owner of its own neither stop, exit, archive nor start a pass.
- D4: with no record, a `done` one, or the subject's owner write recorded, the watcher and the dobby act on a foreign owner as today and as D11 says.
- D11: an old owner's dobby that reads a foreign owner owner-pauses its runs, pushes an `archive-owner/` tag for each local tip origin lacks under its subject, and exits; a failed archive keeps it up, starting no pass of a run whose owner pause stands, and retries at the next probe.
- D11: `lym beam start` of a subject another machine owns and no adopt covers, with no dobby running, pushes an `archive-owner/` tag for each local tip origin lacks before it refuses; a failed archive is named in the refusal, and a re-run retries it.
- D11: the run report lists no run for a D11 tag of a live run or of `root`.
- D11: an idle dobby's periodic Bolt pull brings in a new owner, which its next probe acts on.
