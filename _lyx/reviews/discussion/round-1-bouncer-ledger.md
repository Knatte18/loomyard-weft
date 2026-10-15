---
round: 1
ledger:
  - key: work-dir-clear-unbounded
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: lyx-literals-pattern-departure
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: trace-pack-layout-arm-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: batch-resolution-cross-run
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: runs-name-match-and-charset
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: code-required-under-plan
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: row-sort-tie
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised seven findings against the discussion pair: two BLOCKING design findings (unbounded `--work` clearing; `_lyx` literals departing from PATTERN without amending it), two MEDIUM design findings (unbounded trace `pack_layout` arm; batch resolution searching every ref), one LOW design finding (`--runs` match rule and character set), and two consistency findings (`--code` required under `--plan`; sort-order tie).
All are open; none has been fixed yet.
