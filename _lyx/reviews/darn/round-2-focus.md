---
round: 2
focus:
  - "In internal/fabricengine/archive.go, the comment on archiveTagPushDelays and the matching Design decisions sentence in _lyx/landing/summary.md: settle whether they state accurately which lock a teardown holds while the archive-tag push retries, given Remove takes the pair's teardown lock exclusive (internal/fabricengine/remove.go) before calling archiveWeftTip, and check what Cleanup (internal/fabricengine/cleanup.go) holds when it calls archiveWeftTip."
  - "In internal/fabricengine tests, settle whether a test pins the archive push's schedule (archiveTagPushDelays, 2s doubling to 32s) and its use at the PushWithRetry call in archive.go, so that reverting the call to PushBackoffDelays() or changing the delays fails a test."
  - "In internal/gitrepo/pushretry_test.go, settle whether the PushWithRetry table covers an empty delays slice (one attempt, then ErrPushRetriesExhausted) and whether any added row duplicates an existing row's coverage."
---

The three entries restate the sites of the round-1 findings so a fresh reviewer can check the fixed tree against the board entry.
