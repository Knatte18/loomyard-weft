---
round: 2
focus:
  - "internal/testkit/tmuxkit/tmuxallowed.go: tmuxAllowed reads the run-time tier selection (LYM_TEST_TIER) instead of a build tag. Settle whether a test in the package asserts its result for unset, single-tier, range and malformed values."
  - "crucible/orchestrator-prompt.md smoke commands (go test with -tags tier2, go test -c, and the loop running the compiled binaries): settle whether each run selects tier2 or tier3 through LYM_TEST_TIER, so no smoke test skips green under the run-time selector."
  - "internal/battencli/testmain_integration_test.go: the TestMain is the only one in the package after the tier0 companion was removed. Settle whether a tier0-only selection still refuses a real pair push through the push seam."
---

Focus targets the places where the move from build tags to a run-time selector could leave tests silently skipped or unpinned.
