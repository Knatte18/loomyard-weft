---
round: 2
focus:
  - "internal/driftcli/capture_integration_test.go and internal/driftcli/cli.go (startMainCapture, the child's OnEnd hook): does any capture-failure subtest depend on whether the capture record reads running or failed when the verb reads it, and is the child's non-zero exit and error envelope pinned deterministically?"
  - "internal/driftengine/steprun.go (commandReport, boundOneLine): does the quoted log tail keep the log's final line when the last three lines exceed the output bound, and is the log read without loading it whole?"
  - "internal/orchcli/reconcilenotifier.go and the root pre-run in cmd/lym/main.go: do tests show the notifier mailing the hub's prime orch as RECONCILE, declining another hub's board dir, installing nothing outside a hub, and being installed by the root pre-run?"
---

The focus lists the places whose fixes are new text for a fresh reviewer to read.
