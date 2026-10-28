---
round: 1
focus:
  - "internal/gatecli/run.go and tierrun.go: does the resource-shortage retry fire only when the tier failed, did not time out and was not cancelled, and run at most once per tier per call, inside the slots already held, with the retry's result becoming the tier's result?"
  - "internal/gatecli/tierrun.go: are the three markers matched case-insensitively, and does the closing failed-runs line label a twice-failed marker run as a resource shortage with the way forward, without marking the verdict transient?"
  - "internal/gateslot/config.go, tiers.go, and gatecli/cli.go: does an unset tier or build dir stay empty until WithDefaultDir fills it with the anchor relative to the worktree root, so a set dir stays root-relative, a root anchor yields `.`, and no reader of the dir (green record module key, dotnet scoper, vet and guard runs) still sees an unfilled or wrongly joined path?"
  - "internal/proc/buildserver.go, gatecli, and reedengine/panebin.go: do both build-server variables reach every tier, build, vet and guard run overriding an inherited value, while a tier's own env can still override them, and does the pane launch line export them with correct quoting beside LYM_STRAND_NAME on every shell flavor reed supports?"
  - "internal/loomshed/verifygate.go: does verifyFailureFindings list parsed failures with their own output tail under `Failures:` in place of the log tail, bounded in bytes, and keep the log tail only when ParseVerifyFailures names none; is the byte bound applied without cutting mid-rune or dropping all failures?"
  - "Docs and PATTERN rules: do gatecli, reedengine, proc, loomshed and gateslot doc.go and docs/dotnet-hub.md describe the new behavior, and does the retry log through internal/logger with its delay injected through testRequest (PATTERN-spawn-observability, PATTERN-test-speed)?"
  - "Test economy: are the new tests in tierrun_test.go, test_integration_test.go, verifygate_test.go and panebin_test.go each covering a behavior no existing test covers, and does every acceptance criterion in the decision record have coverage?"
---

Seed focus from the board entry and decision record; round 1 reads the diff over the review range against each acceptance criterion.
