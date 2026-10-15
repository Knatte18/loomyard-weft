---
round: 1
focus:
  - "Card 17 diff in `internal/creelengine`: settle whether plain `read` returns the oldest unread message plus every other unread message from that same sender, and nothing else."
  - "Settle whether `read --from <sender>` returns every unread message from that sender and no read ones, and whether `read --all` returns every unread message and no read ones."
  - "Check that a message returned by `read` is marked read in the journal fold, so a repeated `read` does not return it again."
  - "Check test coverage of the three read sets against the board's D8 acceptance criterion, and apply test economy: no duplicate coverage of an existing read test."
---

Seed focus for round 1, drawn from the plan overview's stated reason for the earlier rejection.
