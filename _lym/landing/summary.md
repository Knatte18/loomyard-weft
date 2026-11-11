# adopt: add --here behind lym s, and --archives for old-owner archives

`lym s` now runs `lym adopt --here --no-orch` before `beam start` and `reed attach`.
A machine that does not own the worktree's subject adopts it first, from a plain shell, and then starts the beam.
A machine that owns it ends with the outcome `owned_here` and goes straight on to `beam start`.
The new `lym adopt --archives <subject>...` lists the archives an old owner pushed after a handover, which the new owner has to read after the adopt ends.

## What changes

`lym adopt --here` adopts the subject of the worktree it runs in: `root` in the prime, the beam in a beam.
It reads the adopt record first.
An unfinished record takes the worktree up when its `check` step is not done
and its subject set lists the worktree's subject, or the subject the worktree reads its owner from through `hubgeom.ParentOf`.
The call then takes the record's request as its own, whoever the owner reads as, so it follows a live worker or resumes the record.
A record past `check` that stopped on a held-back project takes up no worktree, so its other subjects still reach `beam start`.

With no record taking the worktree up, `--here` pulls Bolt through fabric and reads the subject's owner from it.
When origin is unreachable it reads the local Bolt and puts the reason in the envelope's `warnings`.
If no other machine owns the subject, the call ends ok with `owned_here`, spawns no worker and writes no record.
Otherwise it is `lym adopt <subject>`, under the same rules as that call.
In a worktree that is neither the prime nor an open beam project it ends ok with `no_subject`.
`--here` with a subject, `root` or `--all` is refused.

`lym adopt --archives <subject>...` takes beam projects or `root`.
For each slug under the named subjects it reads the `archive-owner/` tags from each repository's origin
and lists those whose commit the slug's branch on origin does not contain: the code branch for code tags, the weft branch for weft tags.
Each listed tag carries a `git -C <dir> log <tag>` command to look at it.
Teardown and `--drop-local` archives carry the `archive/` prefix, so none of them are listed.
The call runs no step, brings nothing in, and reads and writes no record.
It refuses every other flag, an unknown slug, and an entry that is not a beam project, naming the subject that moves a task or grouping project.
The end envelope of an adopt whose owner write stood names this call, because the old owner archives only after it reads that write.

## Design decisions

- `lym s` stops at the first argv that exits non-zero, so a stopped adopt runs no `beam start`.
  `beam start` starts the orch, which is why the shortcut passes `--no-orch`.
- An owned-here `lym s` never meets another subject's unfinished record, because the record only takes up a worktree whose subject it lists.
- Every `lym s` with no record taking its worktree up pays one Bolt pull more than before.
  A pull that cannot reach origin reads the local Bolt, so a stale Bolt that reads this machine as owner skips the adopt.
  `beam start` then judges the owner on the local Bolt too and starts the beam here, as `lym beam start` does today.
  The next Bolt pull that reaches origin hands the subject to the old-owner path.
- `fabricengine` gains the archive-owner tag read, built on a shared `remoteTagsUnder` helper that `remoteArchiveTags` now also uses.
  Git still runs only in `internal/fabricengine`.
- `gitrepo.RemoteURL` wraps a new `ErrRemoteNotConfigured` when the remote is absent or has no URL.
  `fabricengine` uses it so that only a missing origin reads as none, and a failure to read the configuration is an error.
- `adoptrecord.Record.TakesUp` holds the take-up judgment, next to the `Watch` logic it shares with `ownerSubject`.
- `docs/overview.md`, `docs/getting-started.md` and the `adoptcli` package doc describe the new flags and the changed shortcut.

## To check by hand

- `lym s` in a foreign-owned prime and in a foreign-owned beam, against a real second machine and origin.
  The integration tests cover the logic, not a live handover.
- The test that separates records-tag from records-branch archive reads is missing (`records-tag-vs-records-branch-untested`).
  `internal/fabricengine/ownerarchive_integration_test.go` is the place to look.
- No test pins that a teardown archive is left out of the `--archives` list (`teardown-archive-exclusion-untested`).
  The exclusion rests on the `archive/` versus `archive-owner/` prefix split in `internal/fabricengine/ownerarchive.go`.
