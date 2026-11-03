---
verdict: CONTINUE
rationale: "F1 is a design-class MEDIUM finding, so the round is gating; F2 is a LOW consistency finding and gates nothing. No relabel was made."
---

F1 says the comments in `internal/preflight/preflight.go`, `internal/preflight/doc.go` and the summary name a status or records commit as the carrier of the exempt files, while `statusCommitPathspec` and `battenRunCommitPaths` stage neither the settle log nor the stop markers.
The finding's own entry shows the class and severity, so I kept them.
F2 is a stale-doc consistency finding and is not gating.
No finding departs from its labelled class or severity, and the review carries no `## Focus departures` section.
