---
round: 1
ledger:
  - key: assist-seat-escapes-catalog-and-scan
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: seats-func-test-only-caller
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: manifest-omitted-no-production-caller
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: unmet-skip-narrows-d1-unrecorded
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: parity-fixture-lacks-seat-skills
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: card6-refusals-reached-only-by-card8
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: standing-passages-test-repeats-existing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: copy-bound-misstates-start-retry
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: loom-driver-omitted-from-unmet-producers
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: one-way-forward-for-two-copy-failures
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card3-repo-root-source-unnamed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: renderers-named-by-file
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first review; every finding is open.
Three BLOCKING design findings lead:
the burler assist seat escapes both the seat catalog and the scan,
`shuttleparity.Seats` is called only by a test,
and `Manifest.Omitted` has no production caller.
Three MEDIUM design findings follow: the parity fixture holds too few skills, card 6's refusals are reached only by card 8's test, and one new test repeats an existing one.
One MEDIUM decision finding records that skipping seats Codex cannot start narrows D1 with no recorded decision.
The remaining findings are LOW or NIT wording and naming fixes.
