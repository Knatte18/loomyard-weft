# Fixer-report, round 1

## Changes

- F1 (fixed differently): the review proposed reading `UpstreamSHA` first and recording `other` when it fails.
  I did not make that change as proposed: the fix-proposal fork showed it misrecords a branch with no upstream whose name origin already holds, a real divergence, as `other`.
  `failRejection` (`internal/fabricengine/pushclassified.go`) now takes origin's tip from the upstream, or from `refs/remotes/origin/<branch>` when there is none, and compares it with `HEAD` through `CommitsNotIn`.
  It records `other` with the error joined when that tip cannot be read, and `not_fast_forward` only when origin holds commits `HEAD` lacks.
  `HasUnpulled` is no longer called there.
  Added rows: `RecordsWithNoUpstream` in the `ARejectionThatOutlastsTheBackoff…` table (origin lacks the branch, expects `other`), and `ARejectionOfABranchWithNoUpstreamWhoseNameOriginAlreadyHoldsIsADivergence` (expects `not_fast_forward`).
  The package doc in `internal/fabricengine/doc.go` follows.
- F2: added `AJudgmentFetchThatFailsAfterTheFinalRejectionIsRecordedAsOtherWithTheFetchError`.
  A pre-receive hook declines every push and removes the bare's `HEAD` on the last push the backoff allows, so the judgment fetch fails.
  The test expects an `other` record whose error holds the rejection line and `gitrepo: fetch in`.
- F3: `internal/gitrepo/push_test.go` asserts that the `PushRebaseFree` error holds git's line after the sentinel and no `remote diverged`.
- F4: the table's notice check requires the plain-retry way forward in the mail.
- F5: header and test renamed to `TestPushAnchored_DivergedWeftRemote_ReturnsErrPushRejected`, comment updated.
- Gate repair: the F5 header comment in `internal/fabricengine/pushanchored_integration_test.go` was fixed-column wrapped; it is now one sentence per line, with the diverged-weft clause split into two sentences so no line ends in a comma.

Commits: F3, F5, F1+F2+F4 (one test file shared), and a second F1 commit for the same-name case, the `doc.go` line and the F3 assertion.
The tier-1 gate over `internal/fabricengine`, `internal/gitrepo` and `internal/loomcli` passed.

## Self-check

- fix-consistency, LOW: `doc.go` named only the failed fetch as an unknown-origin `other` case. Fixed as proposed.
- fix-code-correctness, LOW: the "rejected" substring in the F3 assertion was always true. Fixed with a different change: the assertion now requires git's line after the sentinel, since the line git prints in this fixture (`hint: Updates were rejected…`) holds no `[rejected]`.
- fix-proposal, MEDIUM: F1's guard misrecords a same-name branch with no upstream. Fixed with my own change (see F1).
- fix-proposal, LOW: renamed test still named in `docs/benchmarks/test-redundancy.md`. Disputed, `premise false`: `cmd/testtiming/redundancy.go:23` only names that file as its output, so the tool generates the document and does not look tests up in it.
- fix-affected-set: no findings.

## Disputed

- fix-proposal LOW on the stale name in `docs/benchmarks/test-redundancy.md`, ground `premise false`; evidence above.

## Deferred

Nothing deferred.
