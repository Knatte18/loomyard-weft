---
round: 1
ledger:
  - key: d6-configreg-membership-unbounded
    rounds: [1]
    status: resolved
  - key: d9-routing-pin-downgrade-unbounded
    rounds: [1]
    status: resolved
  - key: d14-deletions-not-all-duplicates
    rounds: [1]
    status: resolved
  - key: d18-allowlist-no-admission-rule
    rounds: [1]
    status: resolved
  - key: d19-tmuxkit-import-laundering
    rounds: [1]
    status: resolved
  - key: socket-criterion-excludes-live-leak
    rounds: [1]
    status: resolved
  - key: d20-audit-narrative-in-record
    rounds: [1]
    status: resolved
  - key: churn-statistic-rationale-lines
    rounds: [1]
    status: resolved
  - key: notes-restate-decisions
    rounds: [1]
    status: resolved
  - key: d4-garbled-sentence
    rounds: [1]
    status: resolved
---

Round 1 raised six bound and correctness gaps.
Five were unbounded guard downgrades or escape hatches: D6's config membership, D9's routing pin, D14's loomcli deletions, D18's allowlist and D19's `tmuxkit` import ban.
The sixth was an acceptance criterion that could not catch a live leaked tmux server.
It also raised four relocation and wording items: D20's audit narrative, the churn-statistic Rationale lines, Notes that restated Decisions, and D4's garbled sentence.
The fixer resolved all ten in the same round, and I verified each against the current record text.
The round closes with no open entries.
