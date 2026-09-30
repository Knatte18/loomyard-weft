# Webster round 1 — fixer report

All eight findings are fixed, each in its own commit, and none were pushed.
The plan's full `verify:` block passes after the last fix: build, vet, the untagged tests, and the `-tags integration` tests.

## Fixed

- **F1** (`ff4b81084`): `ExcludeAnchoredDir`'s doc comment (`internal/fabricengine/gitexclude.go`) now has one sentence per line, with breaks at the two independent-clause boundaries.
- **F2** (`b3a699536`): `PathTracked`'s first doc sentence (`internal/fabricengine/pathtracked.go`) now sits on one line.
- **F3** (`d7591ee3e`): `WriteConfig`'s two new chain sentences (`internal/vscode/config.go`) each sit on their own line, with a break at the semicolon in the attach-only sentence.
  The legacy lines below them are untouched.
- **F4** (`a54aa5a9a`): the doc comments on `TestExcludeAnchoredDir_AppendsAnchoredLineOnce` and `TestExcludeAnchoredDir_AnchorsToSubpath` are now one line each.
- **F5** (`b17bfb5ee`): `TestInnerRun_OpenErrorIsWarnedNotEscalated` now counts every `level=WARN` line.
  The running arm logs its own stuck warning in both runs, so the test asserts that the failed open adds exactly one warning over the success case's, and that exactly one warning carries the error text and names the slug.
- **F6** (`906544e64`): the new `assertSingleAttachTask` helper checks for exactly one task with label `reed attach`, args `reed attach` and `runOn: folderOpen`.
  Both `TestSpawnDrivenWritesAttachOnlyAndExcludes` and `TestSpawnDrivenOverwritesTasksKeepsSettings` now use it.
- **F7** (`0b8257564`): "an once-marker" is now "a once-marker" in `Call`'s disposition doc comment.
- **F8** (`dd61add11`): `TestExcludeAnchoredDir_AlreadyIgnoredWritesNothing` now stages and commits its `.gitignore` before the call, matching the card's scenario.

## Deferred with reason

None.
