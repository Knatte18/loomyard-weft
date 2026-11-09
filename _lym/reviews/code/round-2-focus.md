---
round: 2
focus:
  - "In internal/fabricengine/failedpush.go, `SyncOptions.notifier` returns `Notifier`, or the installed notifier when `Notifier` is nil. Settle whether a test pins both the precedence of a set `Notifier` and the fallback to the installed notifier, and whether any test that installs the notifier runs serial."
  - "In internal/fabricengine/topology.go, `NewTopology` sets the default gc threshold. Settle whether a test asserts that default without overriding it, since a zero value would pass `gc.auto=0` and disable the removal gc."
  - "Across the eight named fabricengine tests that were made parallel, settle whether any parallel test still reads a package variable that a serial test writes, such as `installedPushNotifier`."
---

Targets the round's fix commits for the notifier and threshold coverage gaps, plus the parallel-safety invariant.
