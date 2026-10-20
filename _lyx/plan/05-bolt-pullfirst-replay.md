# Card 5 — bolt-pullfirst-replay

**Edit:**
- `internal/fabricengine#Bolt.pullFirst`
- `internal/fabricengine#Bolt.divergedDetail`
- `internal/fabricengine_test#TestBolt_PullThenCommitWritten`
- `internal/hubreconcile_test#TestEnsure_UnpushedHubWideCommitOverMovedUpstreamLeavesStampAbsent`
- `internal/hubreconcile/doc.go`

**Uses:**
- `plan:internal/fabricengine#replayBolt`
- `plan:internal/fabricengine#readEndedSet`
- `plan:internal/testkit/boardkit#InstallReplay`

**Intent:** Bolt's pull-first becomes a case of D5's replay.
`Bolt.pullFirst` keeps its fetch, its locks (`board.lock` held by the caller, `.gitrepo-push.lock` taken here) and its fast-forward of a Bolt only behind; where Bolt holds commits the moved upstream lacks it now reads the ended set from the fetched origin Bolt tip and its history (`readEndedSet`, the slugs being those the local commits origin lacks touch) and calls `replayBolt` in place of the seed-commit drop.
A replay `ErrReplayDirty` is the `BoltSkipDirty` skip; Bolt's replay never stops on an entry, so `BoltSkipDiverged` is left only for a cherry-picked non-board commit whose conflict `replayBolt` cannot settle, and `divergedDetail` names the conflicting paths and, as the way forward, settling them in the board worktree and `lyx board sync` (the old `git pull --rebase` text is removed).
A set-aside or a lost value is logged at Warn naming its tag.

`TestBolt_PullThenCommitWritten`'s diverged rows change: a Bolt commit of an unrelated entry under a moved upstream now replays and the writes run; a seed-commit row keeps its drop; a new row has a non-board commit conflicting with origin, which skips `BoltSkipDiverged` naming the path.
`TestEnsure_UnpushedHubWideCommitOverMovedUpstreamLeavesStampAbsent` is re-read against the replay: an unpushed hub-wide config commit over an upstream that moved on other paths is now replayed and the stamp is written, while one over an upstream that changed the same config file still skips with the stamp absent; the test keeps the second row and gains the first, and the kept row's log assertion, today that the divergence Warn names `git pull --rebase`, now asserts the Warn names the conflicting config file's path and `lyx board sync`.
Both tests call `boardkit.InstallReplay` first, since their Bolt replays read the board seam.
`hubreconcile/doc.go`'s sentence that only a resolved `git pull --rebase` in the board dir clears an unpushed hub-wide commit over a moved upstream is rewritten: the pull-first replay clears it, and only a conflict the replay cannot settle keeps the stamp absent.
`internal/orchcli`'s rules fake returns `BoltSkipDiverged` itself and is unaffected, since the value stays; grep the test files for `pull --rebase` and update any assertion on the old detail.

**ImpactSummary:** Bolt.pullFirst's callers (every `PullThenCommitWritten` user: board writes, hub reconcile, orch rules) see fewer diverged skips; no signature changes.

**Commit:** 5: fabricengine: Bolt's pull-first replays by entry
