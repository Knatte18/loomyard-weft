---
verdict: CONTINUE
rationale: "Round 2 carries a BLOCKING finding (F11, card 9's tier1 test reaches tmux against PATTERN-test-speed), and two gating design keys open since round 1 stay open (F12 walk-misses-strict-load-routes, F13 board-rewrite-assumes-restore-and-commit)."
---

The round fails the decision rule's `CONTINUE` condition on F11 alone.
F11 cites `PATTERN-test-speed`, so it stays BLOCKING.

F12 and F13 map to keys first raised in round 1, and both are design-class at MEDIUM.
Neither could be carried into open risks.

No finding is relabelled, and the review carries no `## Focus departures` section.

The review lists defects under `## Not raised` that it held back as outside the incremental round's changed hunks.
They are not findings, so they do not enter the ledger or this verdict.
The review says the first, on `--unset` with two old keys, would be BLOCKING in a full round,
so the round 3 focus asks about it and about the two planned assertions listed there.
