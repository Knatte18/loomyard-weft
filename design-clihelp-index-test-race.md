# clihelp: TestRenderIndex_FormsAndFiltering races on a shared cobra tree (concurrent map writes)

`clihelp-index-test-race` · Note · bug, orch

Imported from [issue #547](https://github.com/Knatte18/loomyard/issues/547).

Seen: pane-inject-safety's post-merge Publish verify (the tmux-tier module-wide run after merging main) crashed internal/clihelp with `fatal error: concurrent map writes` in `pflag.(*FlagSet).AddFlag`, reached from cobra's `mergePersistentFlags` via `clihelp.visibleLocalFlags` (internal/clihelp/index.go:94) and `indexLines` (index.go:47).
The task's diff does not touch clihelp; every other package passed.

Likely cause: `TestRenderIndex_FormsAndFiltering` and its subtests run `t.Parallel()` and walk the same cobra command tree.
`LocalFlags`/`LocalNonPersistentFlags` lazily merge parent persistent flags into the command's FlagSet, which writes a map, so two parallel walks race.

Workaround: `lyx loom goto --to Webster-Burler`, then `lyx loom resume`, so the verify reruns.

Expected: each parallel test builds its own command tree, or the flags are merged once before the parallel subtests start, so the test cannot crash.