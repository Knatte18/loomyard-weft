# hubforge: packed templates, no walk per copy, two-process binding fold

`hubforge.CopyHub` and `hubforge.CopyRegistered` did three costly things on every copy:
a fingerprint walk of the whole template, a copy of every loose git object, and three git processes to fold the relocated `.lym-warp` binding into the board.
This change cuts all three; the copies are about 13 percent of the package's serial test time.

## What changes

- **Packed templates.**
  After the build function and before the fingerprint, `buildTemplateIn` packs every repository under the template root,
  bares and clones alike, with `git repack -d --cruft` and `git pack-refs --all`.
  This holds for the closed shapes and for registered templates.
  A copy then copies a few pack files in place of the loose objects.
  No pack and no `packed-refs` file names a template path, so relocation still rewrites only text files.
- **No fingerprint walk per copy.**
  `copyTemplate` now checks only the poisoned flag (`requireUnpoisoned`).
  `hubforge.SharedHub` keeps the walk before the test (`requireIntact`) and after it in its cleanup, and still poisons the template on a change.
- **Two-process binding fold.**
  `commitRelocatedBinding` drops its separate `git add`:
  `git commit --amend --no-edit -- .lym-warp` stages the file itself, and the push follows.
  The fold stays on git, so HEAD, the branch and the tracking ref keep the reflog entries `internal/fabricengine/reflog.go` reads.

The package doc and the file header of `template.go` describe the packing and the split between the two entry points.

## Design decisions

- **`--cruft` in place of the record's `git repack -ad`.**
  `repack -a -d` packs only reachable objects and leaves unreachable loose objects where they are,
  so a build that leaves any behind would still hand copies loose objects.
  `repack -d --cruft` packs everything reachable into one pack and the unreachable rest into a cruft pack, leaving none loose.
  It needs git 2.37 or later.
- **The bound the record names holds.**
  Without the walk on each copy, a stray write by a `SharedHub` test reaches copies taken before that test's cleanup poisons the template;
  copies taken after fail.
  Today's `SharedHub` sites only read, and this change adds none.

## Tests

- `assertBindingFoldedAndPushed`, used by `TestCopyHub` (a shape) and `TestCopyRegistered_CopyHoldsTheBuildAndIsolatesWrites` (a registration whose build commits):
  the board is clean, HEAD equals its upstream and its branch's tip in the copy's records bare,
  and the tracking ref's newest reflog entry names that HEAD.
  The registration test also asserts the copy's board history is as long as the template's.
- `TestSharedHub_PackedTemplateHoldsNoLooseObject` takes a `SharedHub` of the backend-with-pair shape, which no other test in the package takes,
  so it sees no change since the build;
  it then walks the template and asserts no repository holds a loose object.
- `TestSharedHub` still covers a write poisoning the template at cleanup and a later `CopyHub` failing.
