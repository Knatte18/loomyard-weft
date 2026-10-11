---
round: 2
ledger:
  - key: interrupt-marker-counted-as-working
    rounds: [1]
    status: resolved
  - key: orch-transcripts-in-idle-union
    rounds: [1]
    status: resolved
  - key: weft-default-unbounded
    rounds: [1]
    status: resolved
  - key: span-start-fallback-escapes-clip
    rounds: [1]
    status: resolved
  - key: shedrun-helper-shape-open
    rounds: [1]
    status: resolved
  - key: restated-assumptions-rearged-alternative
    rounds: [1]
    status: resolved
  - key: named-inflight-slug-reads-older-archive
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: local-command-line-starts-working-time
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: foreign-hub-comparison-unspecified
    rounds: [2]
    status: open
    class: decision
    severity: NIT
  - key: crash-sentence-duplicated-in-open-risks
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised six findings; none reappears in round 2, so all six are resolved.
Round 2 raised four new findings, none seen before.
The MEDIUM one is F1, a `decision` finding on the unbounded status read for a named in-flight slug.
The only `design` finding is F2, LOW, on local-command lines counted as working time.
F3 and F4 are NITs.
