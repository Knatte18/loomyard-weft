---
round: 2
focus:
  - "In internal/battenshed/innerrun_publishretry_test.go, check whether every top-level test that builds a retryHarness over t.TempDir(), a fake clock and fake seams calls t.Parallel, and whether any test left serial touches process-global state (logcapture.Capture, t.Setenv, t.Chdir), per PATTERN-test-speed."
  - "In internal/battenshed/land.go, check how stop decides whether to queue the failure notice for a pause versus a failure, and whether that decision depends on the wording of a reason string rather than an explicit signal."
---

Both entries name where to look; severity is the reviewer's call.
