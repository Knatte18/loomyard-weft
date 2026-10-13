---
round: 1
ledger:
  - key: hard-cap-below-one-refusal-lacks-per-layer-way-forward
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: unreadable-run-dir-classed-as-correctness-halt
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised two findings, both open.
The first (F1) is that card 6 gives the hard-below-1 and negative per-perch hard-cap refusals a static `loom.yaml` way forward instead of the per-layer route and `lyx loom resume`.
The second (F2) is that card 8's "circling: hard cap unresolved" row classes an unreadable run directory as a correctness halt, though raw I/O failures belong on a transient row.
