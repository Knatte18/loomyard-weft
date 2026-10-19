# Card 3 — cherrypick-without-empty

**Edit:**
- `internal/gitrepo#Repo.CherryPick`
- `internal/gitrepo_test#TestCherryPick`
- `internal/gitrepo/doc.go`
- `internal/fabricengine#errDropReplayFailed`
- `internal/fabricengine_test#TestBolt_PullThenCommitWritten`
- `internal/fabricengine/doc.go`

**Create:**
- `plan:internal/fabricengine#ErrDropReplayFailedForTest` -> `var ErrDropReplayFailedForTest = errDropReplayFailed`

**Uses:**
- `internal/gitrepo#Repo.CherryPickAbort`
- `internal/gitrepo#Repo.MergeHeadPresent`
- `internal/fabricengine#dropSeedCommits`
- `internal/fabricengine#DropSeedCommitsForTest`
- `internal/fabricengine#BoardDropRequestForTest`
- `internal/gitrepo_test#newRepo`
- `internal/gitrepo_test#writeFile`
- `internal/gitrepo_test#commitAll`
- `internal/gitrepo_test#requireCurrentSHA`
- `internal/gitrepo_test#statusOf`
- `internal/fabricengine_test#newBoltFixture`
- `internal/fabricengine_test#boltFixture.seedCommitLocal`
- `internal/fabricengine_test#boltFixture.moveUpstream`
- `internal/fabricengine_test#boltFixture.head`

**Intent:** D2: `Repo.CherryPick` stops passing `--empty=drop`, which git 2.43 lacks (`cherry-pick --empty` arrived in 2.45), and keeps its result on every git version: a commit whose change HEAD already carries is dropped with HEAD unmoved, and a commit empty from the start is refused as newer git refuses it today.

- `CherryPick` runs `git cherry-pick <sha>` through `runChecked`.
  On success it returns nil.
  On failure it runs, through `runChecked` in the same shape as `MergeHeadPresent`, the stopped-empty probes: a cherry-pick is in progress (`rev-parse --verify --quiet CHERRY_PICK_HEAD`, exit 1 meaning none), no index entry is unmerged (`ls-files --unmerged` prints nothing), and the index equals HEAD (`diff --cached --quiet HEAD`, exit 1 meaning it differs).
  When any condition is false it returns the pick's wrapped error, as today, with the pick left in progress (when one is) for `CherryPickAbort`.
  When all three hold, it compares the picked commit's tree with its first parent's (`rev-parse <sha>^{tree} <sha>^1^{tree}`, two equal lines meaning equal trees): equal trees return an error naming the commit as empty from the start, with the pick still in progress; otherwise it runs `git cherry-pick --skip` and returns nil, so HEAD stays unmoved.
  A failing probe, comparison or `--skip` returns its wrapped error and leaves the pick in progress for `CherryPickAbort`.
  The index-differs return and the failing-probe and failing-`--skip` returns are fail-loud guards in `refusal-spec.md`'s sense: git's plain pick, which is the only argv here, reaches none of them with a stopped pick and nothing unmerged, so they take no reaching test, and a failing probe is a propagated git error.
  Bound: `--skip` runs only on a pick that stopped empty against HEAD with nothing unmerged, so it drops no change and no conflict, and uncommitted working-tree changes (which `dropSeedCommits`' `reset --keep` kept) survive it.
  Every new call goes through `runChecked`, inside `CherryPick`, which `gitrepo`'s `doc.go` already lists as CLI-bound, so no exported method joins the boundary test's list.
  The godoc and `internal/gitrepo/doc.go`'s bullet describe the skip and the empty-commit refusal, and no `.go` comment names the removed flag.
- `TestCherryPick` changes, as rows of its existing step table:
  the already-carried step first dirties a tracked file the pick does not touch (`a.txt`), then asserts nil, HEAD unmoved, the index equal to HEAD (`git diff --cached --quiet`), no `CHERRY_PICK_HEAD` under the repository's git dir, and `a.txt` still dirty, then restores `a.txt` so later steps start clean;
  the conflicting step additionally asserts `CHERRY_PICK_HEAD` exists before the abort;
  a new step "a commit empty from the start is refused with the pick in progress" picks a `commit --allow-empty` commit made on `side` during setup, expects a non-nil error that is not `ErrInvalidSHA` and names the commit's SHA, `CHERRY_PICK_HEAD` present, then `CherryPickAbort` nil and HEAD restored;
  a new step "a pick that never started returns git's error" picks a valid-hex SHA that names no object (`0123456789abcdef0123456789abcdef01234567`), expects a wrapped `*gitexec.GitError`, HEAD unmoved and no `CHERRY_PICK_HEAD`, which reaches the in-progress probe's exit-1 branch;
  a new step "an already-carried root commit returns the comparison error with the pick in progress" picks a parentless commit made during setup with `commit-tree` over HEAD's own tree, expects a non-nil error from the `<sha>^1^{tree}` comparison, `CHERRY_PICK_HEAD` present, then `CherryPickAbort` nil and HEAD restored.
  This host's git 2.53 runs the same plain-pick path git 2.43 does, since `--empty` is gone from the argv.
- `errDropReplayFailed`'s comment and message stop saying the replay conflicted, since an empty-from-start commit now fails it too: "a replay that failed" and "replaying the commits that are not seed commits failed"; no test asserts the old message text.
  `internal/fabricengine/doc.go`'s board seed-commit drop paragraph replaces `cherry-pick --empty=drop` with the plain pick that skips a change upstream already carries and refuses a commit empty from the start.
- `ErrDropReplayFailedForTest`, in `export_test.go`, re-exports the sentinel.
- `TestBolt_PullThenCommitWritten` gains the subtest "an empty local commit fails the replay and restores the original tip": it makes a seed commit, then `git commit --allow-empty -m "local note"` on the board, moves upstream, fetches, and calls `DropSeedCommitsForTest` with `BoardDropRequestForTest(f.board, nil)`, the board's repo, the upstream tip and the two commits ahead (newest first); it expects `errors.Is(err, fabricengine.ErrDropReplayFailedForTest)`, head equal to the pre-drop tip, a clean board, no cherry-pick in progress and no record entry.
  The two Bolt tests that failed on git 2.43 (`TestBolt_PullThenCommitWritten`, `TestBolt_PushRecorded_DropsSeedCommitsOnRebaseConflict`) reach the pick through `dropSeedCommits` and need no other change.

**Verify:** ! grep -rnF -e '"--empty' --include=*.go internal cmd tools

**ImpactSummary:** One production behaviour change in `Repo.CherryPick`, whose sole caller is `dropSeedCommits`; its result is unchanged on newer git, one sentinel's message and two doc paragraphs follow.
