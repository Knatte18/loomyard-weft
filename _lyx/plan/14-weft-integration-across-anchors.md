# Card 14 — The integration that carries a weft across an anchor change

**Edit:**
- `internal/gitexec_test#TestRun`
- `internal/gitexec_test#TestRun_StdoutOnError`
- `internal/gitexec_test#TestCheckFloor`

**Create:**
- `plan:internal/fabricengine#ErrIntegrateConflict` -> `var ErrIntegrateConflict error`
- `plan:internal/fabricengine#IntegrateOutcome` -> `type IntegrateOutcome struct`
- `plan:internal/fabricengine#integrateWeft` -> `func integrateWeft(rec *Mutations, weftPath, upstream, target string, names []string, fallback string) (IntegrateOutcome, error)`
- `plan:internal/fabricengine#resetWeftToIntegrated` -> `func resetWeftToIntegrated(rec *Mutations, weftPath, onto string) error`
- `plan:internal/fabricengine#IntegrateWeftForTest` -> `func IntegrateWeftForTest(weftPath, upstream, target string, names []string, fallback string) (IntegrateOutcome, error)`
- `plan:internal/fabricengine_test#TestIntegrateWeft` -> `func TestIntegrateWeft(t *testing.T)`
- `plan:internal/gitexec#RunWithEnv` -> `func RunWithEnv(args []string, cwd string, extraEnv []string) (string, error)`

**Uses:**
- `plan:internal/fabricengine#treeAnchor`
- `plan:internal/fabricengine#committedMarker`
- `internal/fabricengine#replayWeft`
- `internal/fabricengine#replayRequest`
- `internal/fabricengine#ownedWarpCheckout`

**Intent:**
D10's integration onto a target anchor `T`, which card 15's integrating `--set` runs as its step 4; it is not reachable from any verb before card 15.
`integrateWeft` rebuilds the weft checked out at `weftPath` on its fetched upstream tip `U` (`upstream`, a SHA):
it replays, oldest first, each local commit `U` lacks, skipping each commit that changed `.lyx-anchor`, with each changed path moved from under the anchor its parent commit names (`treeAnchor` of the parent, `fallback` when markerless) to under `T`; paths outside every wired name, and the marker, are replayed unmoved;
then, when `U`'s anchor (`treeAnchor` of `U`) is not `T`, it applies the move from that anchor to `T` on the result: each wired name's tracked content re-rooted and `.lyx-anchor` set to `T`, in one commit.
It builds every commit off the checked-out branch with git plumbing (`read-tree`, `update-index`, `write-tree`, `commit-tree`) over a temporary index file made through `internal/lyxtemp`, never touching the weft's working tree, index or branch: each plumbing command runs through `gitexec.RunWithEnv` with `GIT_INDEX_FILE=<temp index>`, since `gitexec` has no other way to give a local git child an environment variable, and the plumbing stays in `fabricengine`, as the snapshot's `commit-tree` and the fork's `read-tree` already run there through `gitexec.Run`.
`RunWithEnv` is `Run` with `extraEnv` appended to the child environment after the variables `gitexec` sets, reporting every failure exactly as `Run` does.
Only once the result is whole does `integrateWeft` move the branch to it through `resetWeftToIntegrated`, never calling `reset --keep` itself.
`resetWeftToIntegrated` is a `destroy.go` executor with a leading `rec *Mutations`, since `PATTERN-fabric-git` keeps every destroying primitive of `fabricengine` there: it runs the shared pipeline first, then `reset --keep` to `onto`, as the weft replay does before its own reset.
Its request is the weft replay's (`replayRequest` over `ownedWarpCheckout(weftPath)`): containment of the weft worktree, ownership as any worktree of the weft's repository, the prime's main checkout included, since card 17 integrates the prime's weft too, dirtiness as the tracked changes on the paths `onto` rewrites against `HEAD`, and no force.
A replay whose paths conflict stops with the weft unchanged and returns `ErrIntegrateConflict` naming the conflicting local commit and its paths, with the way forward: tag the local tip by hand (`git -C <weft> tag archive/<slug>/<tip> <tip>`), save the changes of the conflicting commit and of every later local commit outside the weft, reset the weft branch by hand to the conflicting commit's parent (`git -C <weft> reset --keep <commit>^`), re-run `lyx fabric anchor <slug> --set <T>`, then write the saved changes back at `T` and commit them; the tag keeps every dropped commit reachable until they are back.
`IntegrateOutcome` reports the new tip, the replayed and skipped commits.
`IntegrateWeftForTest` is the `export_test.go` shim the test drives.

`TestIntegrateWeft` (tier1, a records repo with a bare origin built with `gitkit`) covers: `U` re-anchored from `a` to `b` while the local branch holds two records commits at `a`, integrated onto `b`, puts both commits on `U` with their paths under `b` and the marker `b`; integrating onto `c`, which neither side names, also moves `U`'s content to `c`; a local commit that changed the marker is skipped; a markerless local parent tree re-roots from D5's value; a conflicting local commit stops with `ErrIntegrateConflict` naming it and its paths, the branch, index and working tree unchanged;
a tracked change in the working tree on a path the result rewrites refuses at the dirtiness gate, with the branch unchanged.
`TestRun` gains a subtest beside its environment rows: `RunWithEnv` running `git var GIT_AUTHOR_IDENT` with `extraEnv` setting `GIT_AUTHOR_NAME` returns that name in its output;
`TestRun_StdoutOnError`'s `forms` map gains `RunWithEnv`, so its non-zero exit is recovered as `*gitexec.GitError` with its stdout;
`TestCheckFloor`'s row that a below-floor git refuses every entry point gains `RunWithEnv`.

**ImpactSummary:** `gitexec` gains one entry point beside `Run`, which no existing caller changes to; the edited `gitexec` tests only gain `RunWithEnv` entries and one subtest, and the rest is new `fabricengine` code no verb reaches before card 15.
