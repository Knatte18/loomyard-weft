---
round: 1
focus:
  - "internal/loomcli/redundancynote.go: does the match use `gatecli.RedundancyReportMatches` against HEAD so it holds after a commit touching only `_lym/` and when the diff changes a `tier2` test, and does a red-tier report always give the no-report line naming the gate key?"
  - "internal/shedadapters/rubric.go and bouncer.go: is the marker filled only for a rubric carrying it, for both the Code-Burler-Review and Code-Bouncer call sites, and what happens when `Env.RedundancyReport` is nil or the marker is absent or repeated?"
  - "contracts/stencils/loom/loom-rubric-code-review.md: does the test-economy text tell the reviewer to check a contained test's assertions against the covering tests before filing a fold, delete or `//testtiming:keep` finding?"
  - "pattern/PATTERN-test-speed.md and PATTERN.md: does the index line carry the declared text, and does the background state the package-scoped verdict with the module-wide column, the map, `lym gate testmap`, the redundancy step, their bounds and the gate-finding-as-evidence reading?"
  - "New tests in internal/loomcli, internal/shedadapters, internal/shedrecipe and contracts/stencils/rubric_test.go: does any duplicate an existing test's coverage or assert static stencil prose?"
  - "Scope: do the changes to docs/overview.md, internal/testkit/envkit/envkit.go and internal/gatecli/redundancy_integration_test.go fall under the board entry's Delivers line?"
---

Seeded from the decision record's D8, D9, acceptance criteria and the declared PATTERN change, read against the diff stat and commit messages.
