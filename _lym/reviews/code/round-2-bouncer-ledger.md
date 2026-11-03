---
round: 2
ledger:
  - key: exempt-files-not-carried-by-named-commit
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: settle-records-and-runs-root-docs-stale
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: change-description-omits-settle-record-staging
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised a design finding that the exempt run-state files were not carried by the commit the comments named, and a stale-doc finding on the `shedrun` path comments.
Round 2 shows the code stages the settle log in both status commits, so what remains for the first key is wording: the comments in `internal/preflight` name a pair-teardown commit that does not exist.
The `shedrun` path docs still enumerate a consumer list and overstate how the settle log is staged.
The new key covers `_lym/landing/summary.md`, which omits the settle-record staging and names the wrong records commit.
All three are local text corrections whose result the findings spell out.
