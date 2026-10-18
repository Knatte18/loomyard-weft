---
round: 2
focus:
  - "internal/orchengine/watcher.go readMailbox: when the handler ran and the read acknowledgement then fails, does the state saved by creelFailed keep the handler's phase and queued steps, and is the same reply never acted on twice?"
  - "internal/orchengine/watcher.go retryFreshLaunch: when the withdraw has removed the resume line and ClearResumeNoticed then fails, can a later tick still clear the noticed mark, and does the failure stay a retryable failure rather than failing the whole tick?"
  - "internal/orchcli/creelseam.go creelSeam: does a test reach each method's conversion (Inject text versus command with ttl seconds, ReadMailbox, Withdraw, Hold, Release, Post) against the fake creel client in internal/orchcli/fakecreel_test.go?"
---

The entries name where to look; the reviewer judges each against the committed diff.
