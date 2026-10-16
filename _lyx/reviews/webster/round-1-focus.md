---
round: 1
focus:
  - "Card 3 (stall-freshness) and card 9 (ttl-capped-probe-wait): check the freshness bound of twice the probe ceiling and the soonest-inject TTL cap against the card Intent, including the boundary where the reading's age equals the limit."
  - "Card 6 (sweep-confirm-timing): check the 10 s first-miss confirm, the 50 s backoff cap, and that new mail never lengthens the backoff, in the code path and not only in the tests."
  - "Cards 7 and 8 (no-input-box): check that shuttle's idle probe and creel's stall naming read the same probe field, and that no input box is not conflated with a busy or absent pane."
  - "Cards 10 and 11 (startup-gate-seam, llm-smoke-strand): check that the seam stays engine-neutral with no provider-specific wording outside the provider's own adapter, and that the llm smoke strand's own window and pane-on-failure output match the card Intent."
  - "Cards 1, 2, 4 and 5 (journal tail repair, stderr fallback, spawn exit, carried log): check that each fix covers its edge case (a torn last line with and without a newline, a stdout write failure, a daemon exiting by a lost start race versus any other exit, carried records after a fold and in a running daemon)."
  - "Test economy across all cards: a new test that duplicates an existing test's coverage, and behavior the diff adds with no coverage."
  - "Docs in the same commit: each card's `doc.go`, `pattern/PATTERN-daemon-wakeups.md` and `tools/sandbox/SANDBOX-CREEL-SUITE.md` lines match the behavior the card changed."
---

Seed focus from the plan's Card Index; no round has been judged yet.
