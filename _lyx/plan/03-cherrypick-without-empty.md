# Card 3 — cherrypick-without-empty

**Edit:**
- `internal/gitrepo#Repo.CherryPick`
- `internal/gitrepo_test#TestCherryPick`
- `internal/gitrepo/doc.go`
- `internal/fabricengine#errDropReplayFailed`
- `internal/fabricengine#Bolt.PushRecorded`
- `internal/fabricengine#Bolt.dropSeedCommitsAfterConflict`
- `internal/toolfloor/doc.go`
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/gitrepo#Repo.CherryPickAbort`
- `internal/gitrepo#Repo.MergeHeadPresent`
- `internal/fabricengine#dropSeedCommits`
- `internal/gitrepo_test#newRepo`
- `internal/gitrepo_test#writeFile`
- `internal/gitrepo_test#commitAll`
- `internal/gitrepo_test#requireCurrentSHA`
- `internal/gitrepo_test#statusOf`

**Intent:** D2: `Repo.CherryPick` stops passing `--empty=drop`, which git 2.43 lacks (`cherry-pick --empty` arrived in 2.45), and keeps its result on every git version: a commit whose change HEAD already carries is dropped with HEAD unmoved, and a commit empty from the start is refused as newer git refuses it today.

- `CherryPick` runs `git cherry-pick <sha>` through `runChecked`.
  On success it returns nil.
  On failure it runs, through `runChecked` in the same shape as `MergeHeadPresent`, the stopped-empty probes: a cherry-pick is in progress (`rev-parse --verify --quiet CHERRY_PICK_HEAD`, exit 1 meaning none), no index entry is unmerged (`ls-files --unmerged` prints nothing), and the index equals HEAD (`diff --cached --quiet HEAD`, exit 1 meaning it differs).
  When any condition is false it returns the pick's wrapped error, as today, with the pick left in progress (when one is) for `CherryPickAbort`.
  When all three hold, it asks whether the picked commit introduces a change, with `git diff-tree --quiet --root <sha>` through `runChecked`: exit 0 means the commit changes nothing against its first parent, or against the empty tree for a parentless commit, so it is empty from the start, and `CherryPick` returns an error naming the commit as empty from the start, with the pick still in progress; exit 1 means it has a change HEAD already carries, and `CherryPick` runs `git cherry-pick --skip` and returns nil, so HEAD stays unmoved.
  An already-carried root commit therefore takes the skip and returns nil with no `CHERRY_PICK_HEAD`, as `--empty=drop` drops it on newer git.
  Any other `diff-tree` exit, and a failing probe or `--skip`, returns its wrapped error and leaves the pick in progress for `CherryPickAbort`.
  The failing-probe, failing-`diff-tree` and failing-`--skip` returns are fail-loud guards in `refusal-spec.md`'s sense: git's plain pick, which is the only argv here, reaches none of them with a stopped pick and nothing unmerged, so they take no reaching test, and each is a propagated git error.
  The index-differs return is reachable: a pick whose commit step fails, such as on a failing `prepare-commit-msg` hook, stops with `CHERRY_PICK_HEAD` set, nothing unmerged and the index differing from HEAD, so it takes the reaching step below.
  Bound: `--skip` runs only on a pick that stopped empty against HEAD with nothing unmerged, so it drops no change and no conflict, and uncommitted working-tree changes (which `dropSeedCommits`' `reset --keep` kept) survive it.
  Every new call goes through `runChecked`, inside `CherryPick`, which `gitrepo`'s `doc.go` already lists as CLI-bound, so no exported method joins the boundary test's list.
  The godoc and `internal/gitrepo/doc.go`'s bullet describe the skip and the empty-commit refusal, and no `.go` comment names the removed flag.
- `TestCherryPick` changes, as rows of its existing step table:
  the already-carried step first dirties a tracked file the pick does not touch (`a.txt`), then asserts nil, HEAD unmoved, the index equal to HEAD (`git diff --cached --quiet`), no `CHERRY_PICK_HEAD` under the repository's git dir, and `a.txt` still dirty, then restores `a.txt` so later steps start clean;
  the conflicting step additionally asserts `CHERRY_PICK_HEAD` exists before the abort;
  a new step "a commit empty from the start is refused with the pick in progress" picks a `commit --allow-empty` commit made on `side` during setup, expects a non-nil error that is not `ErrInvalidSHA` and names the commit's SHA, `CHERRY_PICK_HEAD` present, then `CherryPickAbort` nil and HEAD restored;
  a new step "a pick that never started returns git's error" picks a valid-hex SHA that names no object (`0123456789abcdef0123456789abcdef01234567`), expects a wrapped `*gitexec.GitError`, HEAD unmoved and no `CHERRY_PICK_HEAD`, which reaches the in-progress probe's exit-1 branch;
  a new step "an already-carried root commit is dropped" picks a parentless commit made during setup with `commit-tree` over HEAD's own tree, and expects nil, HEAD unmoved and no `CHERRY_PICK_HEAD`, matching `--empty=drop`;
  a new step "a pick whose commit step fails returns git's error with the pick in progress" installs an executable `prepare-commit-msg` hook that exits 1 in the test repo's hooks dir, picks a commit with a change, and expects the wrapped `*gitexec.GitError`, `CHERRY_PICK_HEAD` present, then `CherryPickAbort` nil and HEAD restored, and removes the hook so later steps run without it.
  This host's git 2.53 runs the same plain-pick path git 2.43 does, since `--empty` is gone from the argv.
- `errDropReplayFailed`'s comment and message stop saying the replay conflicted, since an empty-from-start commit now fails it too: "a replay that failed" and "replaying the commits that are not seed commits failed"; no test asserts the old message text.
  The same rewording reaches `Bolt.PushRecorded`'s godoc ("when the drop refuses or its replay fails") and `internal/fabricengine/doc.go`'s two Bolt paragraphs: the pull-first skip list's `diverged` reasons name "a failed replay" in place of "a replay conflict", and the Push retry paragraph says "its replay fails" in place of "its replay conflicts".
  `Bolt.dropSeedCommitsAfterConflict`'s godoc says "a failed replay" in place of "a conflicting replay", and the executor paragraph of `doc.go` says a replay that fails aborts the cherry-pick and restores the tip, and that a non-seed commit is replayed or, when its replay fails, kept at its original tip.
- `internal/toolfloor/doc.go`'s audit records `cherry-pick --empty=drop` as replaced by this card, leaving the two-format `#{S:}` loop as the one form still to be replaced, by card 8, beside the optional `allow-set-title` that card 2 records as kept.
  `internal/fabricengine/doc.go`'s board seed-commit drop paragraph replaces `cherry-pick --empty=drop` with the plain pick that skips a change upstream already carries and refuses a commit empty from the start.
- No Bolt test is added: `dropSeedCommits` handles a failed replay (abort, restore of the start tip and `errDropReplayFailed`) in one branch for every `CherryPick` error, which `TestBolt_PullThenCommitWritten`'s subtest "a replay conflict restores the original tip" pins; with `TestCherryPick`'s empty-from-start step, that test meets the acceptance criterion.
  The two Bolt tests that failed on git 2.43 (`TestBolt_PullThenCommitWritten`, `TestBolt_PushRecorded_DropsSeedCommitsOnRebaseConflict`) reach the pick through `dropSeedCommits` and need no other change.

**Verify:** ! grep -rnF -e '"--empty' --include=*.go internal cmd tools

**ImpactSummary:** One production behaviour change in `Repo.CherryPick`, whose sole caller is `dropSeedCommits`; its result is unchanged on newer git, one sentinel's message, `PushRecorded`'s godoc and the Bolt doc paragraphs follow.
