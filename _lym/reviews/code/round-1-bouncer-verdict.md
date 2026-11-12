---
verdict: CONTINUE
rationale: "F1 is a design-class MEDIUM finding (detached upkeep pass writes to a pipe only the dobby reads, so SIGPIPE can kill it), which is gating; F2-F5 are scope or consistency findings and not gating. No relabels."
---

Round 1 carries one gating finding: F1 (design, MEDIUM) in `internal/beamcli/dobbydeps.go`.
The other four findings (F2 MEDIUM scope, F3 LOW consistency, F4 and F5 LOW scope) are non-gating but still get fixed.
No finding was relabeled, and the review has no `## Focus departures` section.
