---
round: 4
focus:
  - "In `internal/loomcli`, `integrationWriteStubDriverScript` (integration_driverbootstrap_test.go) and `writeStubDriverScript` (smoke_driverstrand_test.go) build the stub provider for runs where `Claude.Prepare` returns `RecordsSessionStarts: true`, so startup is confirmed only by a session-start line in the events file. Settle whether each stub appends such a line before it prints the ready box, and whether `TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver`, `TestSmokeDriverStrand_ReentrantAcrossThreeBootstraps` and `TestSmokeStart_NeverAttachesAndAddsNoOperatorStrand` can now pass."
  - "Find any other fake provider that launches through shuttle with the `claude` config key pointed at a script (for example under `internal/orchcli` smoke files) and settle whether it reaches the same startup path without recording a `SessionStart` line."
---

The focus targets the Publish failure fix; the three failing tests are the check.
