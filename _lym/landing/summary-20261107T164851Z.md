# CopyHub in fabricengine and adoptcli fixtures; hook without dirname

Two cuts to test time in `internal/fabricengine` and `internal/adoptcli`.

## Fixtures copy the template

`newRecordsFixture` in `internal/fabricengine/replay_integration_test.go` now takes `hubforge.CopyHub(t, hubforge.Shape{Anchor: "."})` in place of `hubforge.NewHub(t, ".")`.
That fixture backs every row of `TestRecordsReplay`, the `EndedSet` rows included.
Every `hubforge.NewHub` site of `internal/adoptcli` makes the same switch:
`TestAdoptIntegration_SecondCloneTakesOver`, `TestCommand_RunsFromABeam`, `TestCheckCmd_FromABeamExitsZeroWhileOneRuns`, `TestHubCheckDeps_StaleStampRunsTheTunedFinalizeTiers` and the rows of `TestReadDeployState`.
A copy costs about 17 ms against about 103 ms for a fresh `CloneAndWire`.

No `NewHub` site is kept in either package, since none of these tests is about the clone or the wiring.
The takeover test still gets its second machine from `hubforge.CloneSecond`, which clones fresh from the first hub's bares, so the clone it exercises is unchanged;
`CloneSecond` from a copied hub is already in use in `internal/fabricengine/hubpull_integration_test.go`.
The `internal/hubreconcile` sites are left to the separate `internal/hubreconcile` task, which edits the same file.

## The hook spawns no dirname or basename

`internal/fabricengine/post-checkout.sh` splits the worktree root with `${WORKTREE_ROOT%/*}` and `${WORKTREE_ROOT##*/}`.
`git rev-parse --show-toplevel` answers an absolute path with no trailing slash, for which these agree with `dirname` and `basename`.
The chain wrapper in `chainedHookPreamble` (`internal/fabricengine/hook.go`) takes its directory as `${0%/*}`, falling back to `.` when `$0` holds no slash.
The hook runs on every checkout of a test's pairs, so this saves two processes per checkout.

A hub that already holds the hook keeps its old script, which still works: the installer leaves a hook carrying its sentinel alone.

## Tests

The drift-warning tests (`TestInstallPostCheckoutHook_WeftResolution_Prime` and `_Child`) already cover the warning on a mismatch and the silence on a match through real checkouts.
`TestInstallPostCheckoutHook_ChainIdempotent` gains a checkout after the install, and asserts that the wrapper finds and runs the executable `post-checkout.user` beside it.
No earlier test ran the chained user hook.
