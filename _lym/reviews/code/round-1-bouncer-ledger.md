---
round: 1
ledger:
  - key: webster-fixer-await-relative-run-records
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: restarted-driver-interrupted-fixer-not-replaced
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: judge-start-error-spends-bounce
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: merge-danger-blast-radius-too-narrow
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: interrupted-note-denies-replacements
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: replacement-spawn-error-dropped
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: orphan-sweep-deletes-interrupted-record
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: replacement-markspawned-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: box-disagreement-warn-every-poll
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: fixverifyawait-doc-old-order
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: missing-record-and-plain-capture-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: wrapped-prompt-and-empty-starterror-untested
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 opened every key above.
The gating ones concern the interrupted-fixer path (webster's wait lacking the shuttle config and anchor root,
and a restarted driver's exit paths not replacing the fixer) and the judge's start-error path.
The rest are a narrow blast-radius line, stale or wrong wording, a dropped spawn error, record retention,
log noise and test gaps.
