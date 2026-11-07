# fabricengine tests: anchor and origin fixtures onto registered templates

The anchor-change and origin-reconcile tests of `internal/fabricengine` now copy registered `hubforge` templates instead of building their hub on every call.
Each template is built once per test binary; a test takes its own copy through `hubforge.CopyRegistered`.
The change is test-only, and no assertion changes.

## What changes

- `anchorTemplate` holds the "." hub whose prime warp tracks `sub/` and `other/` in two unpushed commits.
  `newAnchorHub` returns a copy of it.
- `originTemplate` holds the "." hub with the prime's records branch published.
  `newOriginHub` returns a copy of it, with the branch names read from the copy by `originHubOf`.
- `originTaskTemplate` adds the pair "task" with its code branch's upstream set, as `originPair` does.
  `newOriginTaskHub` returns a copy with the pair's code worktree and branch.
  It replaces every `newOriginHub` plus `originPair(t, hub, "task")` site; sites that add other slugs keep `originPair`.
- `movesTemplate` adds the "task" run record at the prime naming Land and publishes the pair's records branch.
  `newMovesPair` returns a copy of it.
- `endedTemplate` adds "task" with its run record and publishes a board holding it as a live entry.
  `endedHub` takes no slugs now: it copies the template and runs `originBoard` on the copy,
  since that push into the records bare goes through a clone outside the hub and leaves the board behind its upstream.
  The one test that ended two pairs builds them on a `newOriginHub` copy through `arrangeEnded`, the template's own build step.
- `originPair`, `writeStatus`, `publishBoard`, `publishBranch` and `boardFileOf` take `testing.TB`, so build functions can call them.

## Design decisions

- Each build function holds the former fixture's steps in the same order, so every start state is the same by construction.
  No build calls `os.Setenv`, `os.MkdirTemp`, `git config --global` or a seam setter, and none keeps the TB past the build.
- `TestReconcileOrigin_Judge`, `_Moves` and `_Ended` keep `boardkit.InstallReplay` at their top, before the copies.
  No build reaches a Bolt replay: the replay runs only on a board both ahead of and behind its upstream,
  and every build leaves the board at its upstream tip, which the template build itself checks.
  The hooks these tests install per subtest, `SetOriginCallHookForTest` and `SetAnchorFailHookForTest` among them, still follow the copy.
- Only slug sets that repeat at least three times got a template, so the "first"/"second", "stopped"/"active"/"unrecorded", "mine" and "plain" sites still add their pairs in the test.

## To check

- The `internal/fabricengine` tier-1 tests were not run in this change; the verify gate runs them.
