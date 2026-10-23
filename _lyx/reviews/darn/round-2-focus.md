---
round: 2
focus:
  - "In internal/orchengine/watcher.go, tickHandoff times the handoff-requested phase out from the creel read of its note request (RequestReadAt via MailReadAt). Settle whether a failed MailReadAt lookup is recorded through creelFailed and holds the timeout, and whether a test in internal/orchengine makes the fake creel fail the lookup and asserts that."
  - "In internal/orchengine/watcher.go, the unpausedSince discount applies to the window after the read. Settle whether a test opens a hub pause after the read and would fail if the discount were replaced by a plain now.Sub(readAt)."
  - "RequestReadAt is cleared at toIdle, enter, the idle cycle start and ResetForFreshLaunch. Settle whether the tests pin the clearing at each site, and whether the sentence in internal/orchengine/doc.go on the handoff timeout matches the code, including a request not yet posted (RequestID zero)."
---

Round 1 left test-coverage and doc gaps around the read-counted handoff timeout.
These entries only point at where to check that the fixes landed and nothing else regressed.
