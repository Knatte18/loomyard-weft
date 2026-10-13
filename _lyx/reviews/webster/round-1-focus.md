---
round: 1
focus:
  - "Cards 3, 4, 9, 11–15: for each guard row in the overview's D11 audit table, does the diff wrap the named site in `logger.Episode`/`EpisodeSet` with the keying the row states (per pane, per strand, per entry, per path), and is recovery logged at Info exactly once?"
  - "Cards 8, 9, 11, 14: each row marked demote (Warn to Info, Info to Debug, first attempt Info and retries Debug) — does the diff change the level at the named line, and does a retry path still log at Info anywhere?"
  - "Cards 1 and 2: does the test-time stderr discard cover every logger restore path in tests, and do `Episode`/`EpisodeSet` behave as the policy text and `PATTERN-log-levels` state, including reset on recovery and concurrent use?"
  - "Card 5: when the watchdog sees a replaced binary, does it retire and respawn without a loop, and does lock contention exit at Debug only?"
  - "Cards 10 and 12: do draft holds and the busy send error carry the draft kind apart from the draft text, and does no log line still include draft text?"
  - "Cards 16–19: does teardown copy the trace and verify logs before removal, prune at 30 days, and does every landing queue a notice that the orch stencil routes to After landing?"
  - "Test economy across all cards: for each new top-level test, is there an existing test covering the same behavior, and does any package pass its row in `cmd/lyx/testdata/test-budget.yaml`?"
  - "Docs and PATTERN: do changed behaviors update the owning `doc.go`, and does `PATTERN.md` carry the new log-level invariants in the same commits?"
---

Seeded from the overview's Card Index and D11 audit table.
Entries name where to look; the base diff review and the rubric's `Also flag` dimensions still apply in full.
