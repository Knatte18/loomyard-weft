---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed-differently
    reason: "the proposed wording names the landing's commit and push, but the only guardedCommit caller is pullRequestRecordCommitter (internal/loomcli/landingdeps.go:530), so the comments name the pull request record's commit and push; the guardedCommit doc keeps its generic last line"
  - id: F6
    disposition: fixed-differently
    reason: "no change made: the review's second option, leaving the slackengine commit in the branch; it widens no assertion, so no Loosened-Check trailer is owed, and moving it out would rewrite history"
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
---

## Changes

- **F1:** with git or pushes off, the teardown's `Push` returns nil and `Level` reads true.
  `Worktree-Teardown` is `Stuck` after `maxTeardownPushRounds` pushes that leave origin lacking a tip.
  `battenshed`'s docs and `pattern/PATTERN-pair-remote.md` state both.
  The `PATTERN` entry gained one sentence for the pushes-off exception, which the parent may want to check against the declared text.
- **F2:** the error mapping is `teardownPushError`, with a table test.
  A tier-1 test drives the real push and level read over a pair one commit ahead.
- **F3:** `DobbyRuns` and `hubDobbyDepsStarting` take the pusher start, and both CLI receivers carry `pusherStart`.
  Tests drive the prime tick, the run read and both wired resume closures.
  The choice of the production start in `hubDobbyDeps`, `DobbyViewReader.Read` and `ownerarchive.go` has no test.
- **F4:** tier-0 tests for the lock, `worktreeGone`, `handedOver`, `pauseCovers` and the restart episode.
  A tier-1 test covers the `NewDeps` conditions.
  `runpusher`'s `TestMain` now sets the hermetic git environment.
  The episode test reads the failure count, not the log line.
- **F5:** comment rewording in `internal/loomcli/prepush.go`.
- **F7, F8:** the missing push and prime-takeover rows.
  `newPrimeRunsAhead` now returns its fixture, so a row can end an entry on origin.
- **F9:** the redundant row is deleted, with a `Loosened-Check:` trailer on its commit.

Gates: `lym gate test` passed on the changed packages.
The packages touched by the self-check fixes passed tier 0 and tier 1 after their last edit, and `fabricengine` after its own.
`cmd/lym` passed tier 0.
One tier-0 run failed on the tier lint for a mixed-tier test file of this round, which was split before the commit.
The new tests were not run against the old code.

After the self-check the round's gate refused comment lines that wrapped at a fixed column or reached 120 characters.
They were rewritten, with no code change, in `dobbyreads.go`, `teardownpush_integration_test.go`, `wire.go`,
`dobbydeps_integration_test.go`, `prepush.go` and `seams_test.go`.

The gate then posted its advisory reach note, and the round ends unchanged on it: no test was added.
The functions `.lym/gate/reach.json` marks as reached by no mapped test carry the caveat that an unmapped test may run them.
A grep of the tests found such tests or assertions for `HandOverTips`, the hand-over in `pairteardown` and the deferred and teardown-lock push records.
This round's own rows call `startRunPusher` (through the wired closure), `hubDobbyDepsStarting` and the prime takeover reads.
That reading is by grep and by the tests' text, not by a coverage run.
`stopError.Error` is a one-line message forwarder that no test was seen to call.

## Self-check

All four forks returned.

- fix-consistency, MEDIUM, F5 comments name the wrong guarded sites: fixed as proposed.
- fix-consistency, MEDIUM, pushes-off removal contradicts `PATTERN-pair-remote.md` and `battenshed` docs: fixed as proposed.
- fix-consistency, LOW, push bound missing from `battenshed` docs: fixed as proposed.
- fix-consistency, LOW, `DobbyRuns` doc line too long: fixed as proposed.
- fix-code-correctness, MEDIUM, beam `Runs` seam unasserted: fixed as proposed.
- fix-code-correctness, LOW, wired resume and read starts unpinned: fixed with an own change for the resume closures; the read-side production choice stays untested, as listed under F3.
- fix-code-correctness, LOW, `DobbyRuns` doc states the wrong condition: fixed as proposed.
- fix-proposal, MEDIUM, pushes-off contradicts the `PATTERN` entry: fixed, merged with the consistency finding.
- fix-proposal, MEDIUM, resume closures and beam `Runs` unpinned: fixed as proposed.
- fix-proposal, LOW, push-bound docs stale: fixed as proposed.
- fix-proposal, LOW, real `PushRun` and `TipsOnOrigin` calls untested: fixed as proposed.
  The `teardownPushError` wrap inside the closure is still covered only through the function's own test.
- fix-proposal, LOW, `NewDeps` conditions untested: fixed as proposed.
- fix-affected-set, MEDIUM, `NewDeps` conditions untested: fixed as proposed.
- fix-affected-set, MEDIUM, resume closures unpinned: fixed as proposed.
- fix-affected-set, MEDIUM, present-pair teardown push untested: fixed as proposed, with the same remaining gap on the wrap.
- fix-affected-set, MEDIUM, pushes-off against the `PATTERN` entry: fixed as proposed.
- fix-affected-set, LOW, push-bound docs stale: fixed as proposed.
- fix-affected-set, LOW, beam `Runs` unasserted: fixed as proposed.

## Disputed

## Deferred

## Rulings
