# lym adopt --here behind lym s, and lym adopt --archives

Adopt now starts from a plain shell on the machine taking over, through `lym s`, instead of from the orch.
The new owner can also list the tips the old owner archived after the handover.

## `lym adopt --here` (D10)

`--here` adopts the subject of the worktree it runs in: `root` in the prime, the beam in an open beam project.
In any other worktree it ends ok with `outcome: no_subject` and does nothing.
A subject, `root` or `--all` beside it is refused.

- **Record take-up.** It reads the adopt record first.
  An unfinished record whose `check` step is not done takes the worktree up when its subjects list the subject the worktree reads its owner from.
  That is the beam itself when the record lists it or it records its own owner, otherwise the subject above it through `hubgeom.ParentOf`, up to `root`.
  The call then runs the record's request with its own flags merged in by `mergeRequest`, so it follows a live worker or resumes the record, whoever the owner reads as.
  The new `adoptrecord.Record.TakesUp` makes this judgment, reusing the watcher's owner-subject resolution.
  A record past `check`, such as one that stopped on a held-back project, takes up no worktree.
- **Owner read.** With no record taking the worktree up, it pulls Bolt through `fabricengine.Bolt.Pull` and reads the subject's owner.
  A pull that cannot bring Bolt up to date, an unreachable origin among them, reads the local Bolt and puts the skip's detail in `warnings`.
- **Ends.** A subject no other machine owns ends ok with `outcome: owned_here` and `subject`, spawning no worker and writing no record.
  Any other subject runs as `lym adopt <subject>` under the existing caller and worker rules.

`shortcuts.yaml`'s `s` becomes `adopt --here --no-orch`, then `beam start`, then `reed attach`.
A shortcut stops at the first argv that exits non-zero, so a stopped adopt starts no beam.

`runAdopt` now takes a `callFlags` value (`Discard`, `Here`, `Archives`) in place of the bare discard bool.
These flags pick what the call does, so they stay off `request`, which converts to `adoptrecord.Request`.

## `lym adopt --archives <subject>...` (D12)

- **Fabric read.** `fabricengine.Topology.ReadOwnerArchives` lists the `archive-owner/` tags on the code and records origins.
  It keeps those whose commit the slug's branch on that origin does not contain.
  A code tag is judged against the slug's code branch: the prime's checked-out branch for the prime slug, otherwise `PairTaskBranchOrOrigin`.
  A records tag is judged against the paired records branch.
  A branch origin lacks contains nothing.
  Each listed tag is fetched as a local tag, so its `git log` works on the new owner.
  The `ls-remote --tags` parsing behind `remoteArchiveTags` is now `remoteTagsUnder`, shared by both readers.
- **Verb.** The verb validates each subject (`root` or a beam project on the board).
  A task or grouping project is refused, naming the subject that moves it.
  It places each tag's slug under its subject through `Owners.AdoptSubject`, and a slug the board no longer holds under its parent's subject.
  It prints the tags per slug, each with side, commit, branch and a `git -C <dir> log <tag>` command.
  It runs no step and reads or writes no adopt record.
  `archive/` teardown and `--drop-local` tags never match the prefix.
- **End envelope.** The adopt's end envelope gains `archives`: the `lym adopt --archives` call over the subjects whose owner write stood.
  The old owner archives only once it reads that write.

## Docs

The package docs of `adoptcli`, `adoptrecord` and `fabricengine` cover the new paths.
`docs/overview.md` and `docs/getting-started.md` describe the new `lym s` and the two flags.

## Tests

- `TestRunAdopt` gains `--here` rows covering:
  - owned here, with and without another subject's unfinished record;
  - a pull that could not reach origin;
  - a foreign prime and a foreign beam;
  - a worker killed at `deploy` resumed with no pull;
  - a beam without its own owner following a running `root` adopt;
  - a record past `check` with a held-back project;
  - a task worktree;
  - the refusal of a subject beside `--here`.
- `TestRunArchives` (tier 0) covers subject placement, the per-slug listing and the refusals.
- `TestAdoptFields` pins the `archives` field.
- `TestReadOwnerArchives` (tier 1) runs over a hubforge hub.
  It covers code and records tags the branch lacks, which are listed and fetched locally.
  It also covers a tag the branch holds and a slug the filter refuses, neither of which is listed.
