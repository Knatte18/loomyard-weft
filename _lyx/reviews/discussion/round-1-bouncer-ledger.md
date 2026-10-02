---
round: 1
ledger:
  - key: prosa-exemption-unbounded-on-source-files
    rounds: [1]
    status: resolved
  - key: skip-seam-trusts-agent-writable-record
    rounds: [1]
    status: resolved
  - key: d9-d10-key-on-highest-round-not-live-generation
    rounds: [1]
    status: resolved
  - key: archive-completion-marker-unnamed
    rounds: [1]
    status: resolved
  - key: d1-review-rundir-path-wrong
    rounds: [1]
    status: resolved
  - key: append-only-removal-bound-overstated
    rounds: [1]
    status: resolved
  - key: d12-rationale-deliberation-history
    rounds: [1]
    status: resolved
  - key: auto-mode-assumptions-restate-decisions
    rounds: [1]
    status: resolved
---

Round 1 is the first round, so no ledger carries over.
The reviewer raised eight findings, and the fixer resolved all of them in the same round.

- Four findings were about bounding the new review bypass.
  The `Prosa` exemption now excludes source files.
  The skip seam re-classifies the live cards and states its bound.
  D9 and D10 now share D7's definition of the live generation's round.
  `record.json` is the archive step's completion marker.
- Two LOW findings fixed a wrong path and an overstated guard-removal bound.
- Two NIT findings were relocation or trim findings: the refusal-audit history moved to the support log, and the Auto-mode assumptions were cut down to pointers.

Every entry is resolved.
A later round reopens an entry only if an edit regresses it.
