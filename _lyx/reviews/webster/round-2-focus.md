---
round: 2
focus:
  - "internal/landingshed/fixerrun.go, fixerRun.run and fixerSession.judge: settle whether a run whose gate failed on an earlier turn and whose later turn hangs, dies or times out before the closure runs again ends as timeout or no-report rather than fixed, and whether fixed is reached only over a passed gate result."
  - "internal/loomcli/doc.go, subjects.go, decision.go, validate.go, internal/battenshed/doc.go and notice.go, internal/loomshed/doc.go, internal/testkit/envkit/envkit.go: settle whether any text, help string or fixture entry still names a row, gate or segment no recipe or gate schema has (PR-Gate, PR-Align, Merge-Bouncer, PR-Rework gate, Merge-Review, publish gate list), and whether tests using the envkit fixtures still pass."
---

Targets the branch's own range; the two entries cover the gating design question and the stale-name sweep.
