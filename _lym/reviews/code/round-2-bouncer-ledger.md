---
round: 2
ledger:
  - key: raw-dotnet-test-offered-despite-refuse-raw
    rounds: [1]
    status: resolved
  - key: no-test-line-misses-skipped-classes
    rounds: [1]
    status: resolved
  - key: no-test-notice-missing-rows
    rounds: [1]
    status: resolved
  - key: codelocation-absent-code-dir-no-row
    rounds: [1]
    status: resolved
---

Round 1 opened four findings, all `design`: two MEDIUM and two LOW.
Round 2 found nothing new, and its focus verdicts show each round 1 finding fixed:
- the stencils and `noTestNotice` condition the hand run on `refuse_raw`;
- the no-test line states its limits, and `internal/gatecli/doc.go` lists them;
- `TestRunTierNoTestNotice` has the missing rows;
- `TestCodeLocationReturnsTheLocationUnchanged` pins the absent code directory path.
