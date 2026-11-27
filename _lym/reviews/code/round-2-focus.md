---
round: 2
focus:
  - "internal/webstercli and internal/loomcli: are the changed Go files gofmt-clean, in particular the fabricSync call sites in acceptaudit.go, beginbatch.go, fixplan.go, fixverify.go, rebaseline.go, recordbatch.go, recoverbatch.go, restoreplan.go and run.go under internal/webstercli?"
  - "internal/webstercli/wiring.go and wiring_test.go: do tests fail if wireHub or wireStandalone stops reading FABRIC_SKIP_GIT, FABRIC_SKIP_PUSH or the strand-name variable into syncOptions and strandName, and are any such assertions sound and free of duplicated coverage?"
  - "_lym/landing/summary.md: does each claim, the loomcli timing claims against the table of before and after runs in particular, match the figures and the diff?"
---
