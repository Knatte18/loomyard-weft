---
round: 2
ledger:
  - key: pattern-shed-stop-chain-stale-for-webster
    rounds: [1]
    status: resolved
  - key: run-decision-record-refusal-untested
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: master-none-row-asserts-stencil-prose
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: judges-first-webster-only-marking-unpinned
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
---

Round 1's `PATTERN.md` stop-chain finding is resolved; round 2 found the entry sound.
The decision-record refusal in `Run` was fixed in round 1 by adding a refusal, but the read order makes it unreachable, so the finding reopens as BLOCKING design (F1).
The Webster-only `JudgesFirst` marking still has no test through `goDriverDepsFor` (F2, MEDIUM design).
The master rows still anchor on stencil headings (F3, LOW).
Each fix is local and already fixed by the finding text.
