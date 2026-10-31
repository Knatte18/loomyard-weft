# Fixer report, round 1

## Changes

- F1 (fixed as proposed): `internal/landingshed/prreview_test.go` gains the subtest `CreditOffOpensThePullRequestWithTheDescriptionBodyAlone`, which clears `Config.Credit` and asserts the `Create` call carries `Change body` alone.
- F2 (fixed as proposed): the same file gains `FailingEnginesRunReadIsAnErrorWithNoPullRequestCall`, which fails `EnginesRun` and asserts the error wraps the read error and no pull-request call is recorded.
- F3 (fixed as proposed): `internal/summaryparser/summary.go` renames the `LandingMessage` parameter to `credit` and rewords its doc.
  The self-check found the same stale term in `internal/summaryparser/description_test.go`, so its `TestLandingMessage` field and row names now say credit.
- F4 (fixed as proposed): `internal/loomcli/prcontinue.go` header now says "gate and credit block".

Commits: F1 and F2 share one commit because both edit one file; F3 and F4 have their own.
`lym gate test` over `./internal/landingshed`, `./internal/summaryparser` and `./internal/loomcli` is green; `./internal/summaryparser` is green again after the last edit.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, stale `trailers` field and row names in `TestLandingMessage`: fixed as proposed.
- fix-proposal: no findings.
- fix-affected-set: LOW, the same `TestLandingMessage` member of F3's set: fixed as proposed, duplicate of the fix-consistency finding.

## Disputed

None.

## Deferred

None.
