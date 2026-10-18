---
round: 1
focus:
  - "internal/shedverbs/loop.go: does removing `LoopSpec.LockGrace` leave any caller, fixture or doc (`doc.go`, PATTERN) still referring to it, and does `runLoop` behave identically in production?"
  - "Scope against the board entry: does the sweep of other integration-tier fixtures with short grace or timeout values miss any path expected to succeed (grep the integration-tagged tests for small literal durations), and does any changed bound exceed the entry's license?"
  - "internal/verifytree `TestVerify_SlotGate` and internal/githubclient auth-token timeout test: does the widened bound still catch the regression the test exists for, with the claimed 1.4 s and 5 s reasoning holding?"
  - "Summary claims: check each claim in `_lyx/landing/summary.md` (preflightshed held-lock case keeps 50 ms, lock free-lock row, untagged tests run in the integration tier) against the diff."
---

Seed focus only; the reviewer still reads the full committed diff.
