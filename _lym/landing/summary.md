# hubforge: pack templates, skip the fingerprint walk per copy

`hubforge.CopyHub` and `CopyRegistered` get cheaper.
Copies are about 13 percent of the package's serial time, and the fingerprint walk plus the loose objects are most of each copy.

## What changes

- A template build now packs every repository under the template root with `git repack -d --cruft` and `git pack-refs --all`.
  This runs after the build function and the board-at-upstream-tip check, and before the fingerprint, for shapes and registered templates alike.
  A copy copies a few pack files in place of the loose objects.
  Relocation rewrites only text files, and no pack or `packed-refs` file holds a template path.
- `CopyHub` and `CopyRegistered` no longer take the fingerprint.
  They fail only on a template already marked poisoned (new `requireUnpoisoned`).
  `SharedHub` keeps `requireIntact`, so the walk still runs before the test and again at its cleanup, which poisons the template on a change.
- `commitRelocatedBinding` drops its separate `git add`.
  The amend names the `.lym-warp` binding file, so the fold is two git processes (amend, push) in place of three.
  It stays on git, so HEAD, the branch and the tracking ref keep the reflog entries `internal/fabricengine/reflog.go` reads.
- `internal/hubforge/doc.go` and the function comments describe the new split.

No export is added or changed.

Closes: none (no issue is filed for this task).

## Design decisions

- The repack keeps `--cruft` where the design note says `repack -ad`.
  `-ad` drops unreachable packed objects and leaves unreachable loose ones loose.
  `--cruft` packs them into one cruft pack, so no object a fixture may rely on is lost and none stays loose.
- A stray write by a `SharedHub` test can now reach copies taken before that test's cleanup poisons the template; copies taken after fail.
  Today's `SharedHub` call sites only read, and this change adds none.
- `gitDirsUnder` finds repositories by a `HEAD` file plus an `objects` directory, follows no link and does not descend into a repository it found.

## Tests

- `assertBindingFoldedAndPushed` (used by `TestCopyHub` and `TestCopyRegistered_CopyHoldsTheBuildAndIsolatesWrites`) checks that a copy's board is clean.
  It also checks that the board's HEAD equals its upstream and is the branch tip in the copy's records bare, and that the tracking ref's newest reflog entry names that SHA.
- The registered-copy test also checks the copy's board history is as long as the template's.
- `TestSharedHub_PackedTemplateHoldsNoLooseObject` checks that a `SharedHub` right after the build sees no change, and that no repository in the template holds a loose object or loose branch ref.
  Remote `HEAD` symrefs are excluded, since they stay loose after `pack-refs`.

## To check by hand

- Nothing was run by hand beyond the package's tests.
  Check that the poisoned-template path still fails a later `CopyHub` after a `SharedHub` write; the diff adds no test pinning that criterion.
