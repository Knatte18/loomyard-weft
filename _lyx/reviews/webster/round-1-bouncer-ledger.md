---
round: 1
ledger:
  - key: clone-synopsis-omits-shortname
    rounds: [1]
    status: resolved
  - key: docs-fresh-bind-clone-omits-shortname
    rounds: [1]
    status: resolved
  - key: overview-fabric-bullet-not-one-sentence-per-line
    rounds: [1]
    status: resolved
---

Round 1 is the first round of this Webster-Review pass, which follows PR-Rework's cards 17–20 (the code → shortname rename).
The reviewer raised three findings, and the Burler fixed each in the same round.
F1 (`c0d7f01dd`): the clone synopsis, usage refusal and first help example left out `--shortname`.
F2 (`b89ef2c83`): README, `docs/sandbox-hub.md` and fabric suite F13 showed fresh-bind clones without the flag.
F3 (`bba52bb95`): the fabric bullet in `docs/overview.md` was not one sentence per line.
I checked all three on HEAD and marked them resolved.
Card 18's Move check, the rename's completeness, the unchanged test expectations, the refusal texts and the comment conventions all passed, so none of them became a finding.
