---
round: 2
focus:
  - "In internal/output/summary.go, summaryRenderer.list: for a plain list under a note key (warnings, way_forward, a key ending _error) longer than the collapse limit, settle whether every string is still printed on its own line at any depth rather than reduced to a count."
  - "In internal/output/summary_test.go: settle whether rows pin the five-item and six-item collapse boundary for a plain list, and note keys (an _error key, a scalar _warning, a way_forward) outside a collapsed object at depth zero and one."
  - "In internal/clihelp/readable_test.go and exec.go: settle whether tests cover the finish steps running before the cobra error envelope when a verb leaves an unterminated line and then fails, and a command's own local envelope flag shadowing the global one."
  - "Terminal detection for operator output: settle whether internal/clihelp and internal/shedverbs/human.go now share one terminal-writer check, and that the shedverbs test seam still works."
  - "pattern/PATTERN-cli-cobra.md: settle whether the background describes the global envelope flag installed by clihelp.InstallReadableOutput and its shadowing by a local flag, as it does for the global json flag."
---

Each entry names a place and a question; none states an outcome.
