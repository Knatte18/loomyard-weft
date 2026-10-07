---
round: 1
focus:
  - "Cards 7-9, `internal/verifyrun` and `internal/verifytree`: settle whether the Unix group kill and the timeout actually reach descendants, and whether an expiry is a failed verify (never an error) at every `verifytree.Verify` call site, not only the first one."
  - "Cards 1, 11 and 13: settle whether every wait-mark and wait-marker failure (set, clear, write, remove) is logged and changes no verdict, error return or gate outcome, and whether every marker `lyx loom status` reads carries a pid and treats a dead pid as absent."
  - "Cards 14 and 17: settle whether `lyx loom resume` and batten's revive read reed's sessionless `Directory` and never the seed's driver, whether `lyx loom resume` wakes only a parked driver and refuses everything else with a way forward, and whether batten revives once per episode and never resumes the child."
  - "Card 16: settle whether batten's spawn confirmation is per process, so a restarted batten re-spawns a driverless running child, without double-spawning a child whose driver is alive."
  - "Cards 3, 5 and 6: settle whether `--unless-name` skips over a dormant strand, and whether reed resume drops a non-live strand only when all its done-when paths exist; check shuttle sets an ungated run's output files as done-when."
  - "Per-card mechanical check: for each card, confirm every group's type-specific check from the plan spec ran against that group's own targets, not just the first label's."
  - "Test economy: for each new test, name any existing test that already covers the behavior, and name any behavior the diff adds that has no coverage."
  - "Comment conventions: check new and changed doc comments and `doc.go` text against the repository's `CLAUDE.md`, `PATTERN.md` and the surrounding code, including the docs-in-same-commit rule for the new reed op and the new `lyx loom resume` verb."
---

The review range is `git diff $(git merge-base main HEAD)..HEAD`: `product.parent` is `main` and no `_lyx/loom/rework` round exists.
Entries name where to look and the question to settle; none pre-states a verdict or caps severity.
