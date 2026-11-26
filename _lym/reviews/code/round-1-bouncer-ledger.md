---
round: 1
ledger:
    - key: shellguard-doc-describes-claudeengine-golden
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: mechanical
    - key: settings-golden-path-literal-repeated
      rounds:
        - 1
      status: resolved
      class: consistency
      severity: BLOCKING
      fix: mechanical
    - key: word-scan-fail-closed-end-uncoupled
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: design-wrong
    - key: merge-danger-omits-claudeengine
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
---
Round 1 opened every key (F1 to F4 in the order listed).

The first two cite `PATTERN-code-conventions`: package-doc scope, and one named constant for a path.
The third concerns the word-level exception scan in `internal/shellguard`:
its fail-closed behaviour depends on an end pattern the caller builds, kept in step with the word pattern by hand.
Its fix adds a shared interface for that end, so it is marked `design-wrong`.
The fourth is the blast radius in the change description.
