# Fixer report — plan, round 1

## Changes

- F1 (BLOCKING), card 9: fixed as proposed, with the loop's details tightened by the self-check.
  `awaitMarkedProbe` waits for the marked read, then loops under the real-time deadline until `PausedCalls(receiver)` reaches `mark+2`.
  Each pass polls both that count and the probe-wait count read from `e.clock.Waits()`, waiting for `seen+pass` before advancing 8 s.
  `seen` is taken by the caller together with the mark, after the post returns.
  The bullet states why the loop cannot lose the advance for good and what bounds the total advanced: at most 8 s with one live worker, plus 8 s per wait another live worker records before the receiver parks.
  The helper returns the total advanced and its passes.
  Each negative check asserts that total is shorter than the shortest hold, TTL, back-off or stall freshness it runs under, which replaces the single-8-s claim.
  The Create signature gains `seen` and the return values, and Uses gains `isProbeWait` and `creelfake#Clock.Waits`.
- F2 (MEDIUM), card 3: fixed as proposed.
  The tree comparison is replaced by `git diff-tree --quiet --root <sha>` (exit 0 means empty from the start and is refused, exit 1 means skip, any other exit is a fail-loud guard).
  The root-commit step now expects nil, HEAD unmoved and no `CHERRY_PICK_HEAD`, and the comparison-error claim is gone.
- F3 (MEDIUM), 00-overview: fixed as proposed. `./internal/websterengine` now sits on both `lyx gate test` lines, after `./internal/testkit/creelfake`.
- F4 (MEDIUM), card 8: fixed as proposed.
  The unmarked render drops the current-before-other assertion, giving the reason (tmux 3.4's name order, with `worktree` sorting after `a-very-long-session-name`).
  The prime-first order is asserted only by the marked renders, and the comment no longer claims session-id order.
  The self-check added a rename of the test to `TestBar_LineTwoListsSessionsPrimeFirst`.
- F5 (LOW), card 4: fixed as proposed.
  `Run`, `RunStdin` and `RunGit` are under Edit and gone from Uses.
  The runCore bullet names the floor error as the other non-`*GitError` result in five places: runCore's, Run's, RunStdin's and RunGit's godocs and gitexec.go's file header.
  The self-check added the package summary in `gitexec/doc.go`.
- F6 (LOW), card 3: fixed as proposed.
  `Bolt.PushRecorded` is under Edit, and its godoc and doc.go's two Bolt paragraphs say the replay failed instead of conflicted.
  The self-check added `Bolt.dropSeedCommitsAfterConflict`'s godoc and doc.go's executor paragraph.
- F7 (LOW), card 2: fixed as proposed. `docs/shared-libs/README.md` is under Edit, with a bullet adding `internal/toolfloor` beside `internal/dotgit`, and the ImpactSummary follows.
- F8 (LOW), card 7: fixed as proposed. `CheckTmuxFloor` runs at the top of `RunE`, after `ShouldAbort` and before `EnsureSession`, with the reason replaced and the ImpactSummary following.
- F9 (LOW), card 3: fixed as proposed.
  The index-differs return is out of the fail-loud list and named reachable.
  A new TestCherryPick step drives it through a failing `prepare-commit-msg` hook, and the step removes the hook afterwards.
- F10 (LOW), card 4: fixed as proposed, on the keep-exec.Command branch.
  The probeGitVersion bullet says the probe bypasses runCore only because runCore's first step is the floor check, which the probe would re-enter under the cache mutex, and that it keeps runCore's local-command environment.
- F11 (LOW), card 1: fixed as proposed. The 0700 mode is asserted only off Windows, and the name pattern on every platform.
- F12 (LOW), card 6: fixed as proposed. The bound states that only reedengine's `_test.go` files set `execHook` and `floorLine`, so every production run reaches `CheckTmuxFloor`; no non-test file assigns `execHook`, which I checked.
- F13 (NIT), cards 2, 3 and 8: fixed as proposed.
  Card 2's audit records the two above-floor forms as found and to be replaced, without claiming none newer than the floor.
  Its halt rule exempts those two forms.
  Card 3 records `--empty=drop` as replaced.
  Card 8 adds `internal/toolfloor/doc.go` to Edit and rewrites the audit sentence to the final none-newer-than-the-floor statement.

## Self-check

- fix-consistency, MEDIUM: a pass named `AwaitWaitsWhere` for an either-or wait it cannot do. Fixed as proposed: the pass polls `PausedCalls` and `Clock.Waits` itself.
- fix-consistency, LOW: the test name still claimed session-id order. Fixed as proposed: card 8 renames the test.
- fix-consistency, LOW: card 2's audit list and halt rule disagreed with "to be replaced". Fixed as proposed: both lines reworded, and the halt rule exempts the two forms.
- fix-affected-set, LOW: F6 missed `dropSeedCommitsAfterConflict`'s godoc and doc.go:801-802. Fixed as proposed.
- fix-affected-set, LOW: F5 missed `gitexec/doc.go`'s package summary. Fixed as proposed.
- fix-proposal, BLOCKING: `seen` taken after `awaitPauseRead` could already include the receiver's wait, stalling a lone worker. Fixed as proposed: the caller takes `seen` with the mark after the post, and the signature gains it.
- fix-proposal, LOW: the same `AwaitWaitsWhere` either-or as the consistency finding. Merged into it and fixed there.
- fix-claims, MEDIUM: a wake does not skip recording a wait, since `select` evaluates `clock.After` first (deliver.go:142-144). Fixed as proposed: the no-advance claim is gone, and the bound says at most 8 s and names the wake case.
- fix-claims, LOW: the card 2 sentence read as claiming none newer than the floor. Fixed as proposed: reworded to "makes no none-newer-than-the-floor claim".
- fix-claims, LOW: the doc still called `--empty=drop` "to be replaced" between card 3 and card 8. Fixed with card 3 editing `internal/toolfloor/doc.go` to record it as replaced.

## Disputed

None.

## Deferred

None.
