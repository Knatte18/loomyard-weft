# Card 6 — pull-replay-reflog

**Create:**
- `plan:internal/fabricengine#trackingReflog` -> `func trackingReflog(repoDir, trackingRef string) ([]string, error)`
- `plan:internal/fabricengine#rewriteKind` -> `type rewriteKind int`
- `plan:internal/fabricengine#classifyRewrite` -> `func classifyRewrite(repo *gitrepo.Repo, localTip, originHead string, earlier []string) (kind rewriteKind, dropped []string, err error)`
- `plan:internal/fabricengine#ErrWarpRewound` -> `var ErrWarpRewound error`
- `plan:internal/fabricengine#WeftStopError` -> `type WeftStopError struct`
- `plan:internal/fabricengine#WeftStopError.Error` -> `func (e *WeftStopError) Error() string`
- `plan:internal/fabricengine_test#TestPull_ReflogJudgesUnpushedAndRewinds` -> `func TestPull_ReflogJudgesUnpushedAndRewinds(t *testing.T)`

**Edit:**
- `internal/fabricengine#Fabric.Pull`
- `internal/fabricengine#PullResult`
- `internal/fabriccli#pullResultMap`
- `internal/fabriccli#addWeftVerbs`
- `internal/fabricengine_test#TestPull_WeftDivergedWarpAdvancesCleanly`
- `internal/fabricengine_test#TestPull_WeftDivergedAndWarpFetchFails_PartialError`
- `internal/fabricengine_test#TestPull_AbortsOnUnpushedPlusDiverged`
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#recordsConflictWayForward`
- `internal/fabricengine#rewoundWayForward`
- `internal/fabricengine#replayWeft`
- `internal/fabricengine#readEndedSet`
- `internal/testkit/boardkit#InstallReplay`
- `internal/fabricengine_test#buildReconcileFixture`
- `internal/fabricengine_test#rewriteWarpRemoteHistory`

**Intent:** D9: `lyx fabric pull` brings the weft level or says why not, and its warp arm judges unpushed commits and a rewind by D4's reflog tests.

The reflog helpers serve this card and the reconcile cards alike.
`trackingReflog` returns the values of a remote-tracking ref's reflog from the last entry this machine's own push wrote (git's `update by push` entry) onward, newest last, through `gitexec.Run`; a reflog holding no push entry, as for a branch this machine never pushed, returns every value it holds; a ref with no reflog returns none.
`classifyRewrite` reads, for a local tip and origin's head, the earlier tracking-ref values: **rewound** when some earlier value is not an ancestor of origin's head while origin's head is an ancestor of it and the local tip holds commits of that value origin's head lacks (returning those as `dropped`); **rewritten** when every local commit origin's head lacks is reachable from an earlier value and origin's head is an ancestor of none of them; otherwise none.
The two never overlap: whether origin's head is an ancestor of an earlier value separates them.

`Fabric.Pull`'s weft arm: a fast-forward runs as today; where the weft cannot fast-forward it reads the ended set from the records repo's origin Bolt ref as last fetched and calls `replayWeft` under the weft write lock and push lock.
A replay stop, conflict or dirtiness, no longer warns past: the warp arm still runs, as today, and `Pull` then returns a `*WeftStopError` naming the weft branch, the conflicting paths and the way forward, with the warp result kept: for a conflict, card 1's `recordsConflictWayForward` over the weft worktree with `lyx fabric push` in the pair's code worktree as the push verb, whose either choice leaves the weft level with or ahead of origin so a re-run `lyx fabric pull` replays nothing; for dirtiness, committing the named uncommitted paths with `lyx fabric commit` and re-running `lyx fabric pull`; any git command it names must pass `internal/shellguard`'s rows for agents (PATTERN-no-denied-recovery); it never returns success with `weft_pulled: false`.
The manual `git -C <weft> reset --hard origin/<branch>` recovery is removed from `Pull`'s doc comment, `pullResultMap`'s comment, the `pull` verb's long help in `addWeftVerbs` and `fabricengine/doc.go`'s `Fabric.Pull` description, each of which now describes the replay and the stop.
The warp arm's unpushed test moves after the fetch: a local commit counts as unpushed only when no value `trackingReflog` returns reaches it, replacing the `@{u}..HEAD` read before the fetch.
A task pair's branch `classifyRewrite` reads as **rewound** is refused with `ErrWarpRewound` before any reset, naming the dropped commits and card 1's `rewoundWayForward`, changing nothing; `ResetPairCode`'s `remoteDivergenceWayForward` is not named, since it tells the operator to re-run a reset this machine never ran; the prime's warp branch takes no such refusal and a rewind of it resets as today.
A **rewritten** branch takes the existing anchor walk and reset; a local commit no reflog value reaches still refuses with `ErrWarpDivergedUnpushed`.
`PullResult`'s `WeftPulled` doc is updated to say a weft stop is an error, not a false field.
Bound: the pull discards no non-empty local records commit except an ended entry's, kept under its tag, resets away only commits a reflog value reaches, and never a rewound task branch.

`TestPull_WeftDivergedWarpAdvancesCleanly` becomes the replay row (a weft commit under unrelated origin commits replays and the pull succeeds) plus a conflicting row (a both-sides `status.json` change returns `*WeftStopError` after warp advanced, its message carrying `recordsConflictWayForward`'s exact line, its git steps in the weft worktree and `lyx fabric push` in the pair's code worktree) and a dirty row (an uncommitted change on a path the replay touches returns `*WeftStopError` naming the path, nothing moved on the weft); `TestPull_WeftDivergedAndWarpFetchFails_PartialError` asserts the joined weft stop and warp partial error.
`TestPull_AbortsOnUnpushedPlusDiverged` stays the row for a local commit no reflog value reaches, now judged after the fetch, still refusing with `ErrWarpDivergedUnpushed`; the prime rewind keeps its coverage in `TestPull_ReanchorsSingleCommitBack` and `TestPull_ReanchorsMultiCommitBack`.
Every changed and new pull test calls `boardkit.InstallReplay` first, since the weft arm reads the ended set through the board seam.
`TestPull_ReflogJudgesUnpushedAndRewinds` (integration) carries only what no existing pull test reaches, with rows: after a fetch that already moved the tracking ref to a rewritten head, a branch with no unpushed commit, fetched but never pushed from this clone so its tracking ref's reflog holds no push entry, takes the anchor walk and reset and ends at origin's head with the weft re-anchored; a task branch rewound on origin by a leased update refuses with `ErrWarpRewound` naming `rewoundWayForward`'s exact line and changes nothing, before and after its tracking ref was fetched; a weft with only commits that replay empty pulls clean.

**ImpactSummary:** `Fabric.Pull` has two production callers, `lyx fabric pull` and batten's Land pull, which now see a weft stop as an error; Land's handling follows in card 7.

**Commit:** 6: fabricengine: pull replays the weft and judges warp by the reflog
