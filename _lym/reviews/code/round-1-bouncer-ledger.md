---
round: 1
ledger:
  - key: reflag-after-notice-cleared
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: pullrequestrecorded-seam-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: notice-lost-when-pr-recorded-on-landing-tick
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: landnotice-test-rows-missing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, all `design`.
F1 maps to `reflag-after-notice-cleared`: `flagMembers` can set the flag again once the notice cleared it, giving a second landed line.
F2 maps to `pullrequestrecorded-seam-untested`: the `PullRequestRecorded` closure in `internal/beamcli/dobbydeps.go` has no test.
F3 maps to `notice-lost-when-pr-recorded-on-landing-tick`: a member that lands in the tick the pull request is first recorded gets no line.
F4 maps to `landnotice-test-rows-missing`: `TestLandNotices` lacks rows for the filter, retry and follow-up paths.
