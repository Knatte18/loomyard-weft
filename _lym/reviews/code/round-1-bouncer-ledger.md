---
round: 1
ledger:
    - key: agent-carries-over-live-predecessor-instructions
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: design-wrong
    - key: agent-session-has-no-end-of-life
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: design-wrong
    - key: agent-parent-directive-contradicts-stencil
      rounds:
        - 1
      status: open
      class: consistency
      severity: MEDIUM
      fix: design-wrong
    - key: open-role-normalization-untested
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: agent-verb-untested
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: agent-button-blocks-silently-under-pause
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: design-wrong
    - key: session-spec-test-misses-parent-and-pattern-directives
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: open-help-restates-role-rule-literal
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: merge-danger-blast-radius-incomplete
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: overview-shuttle-verb-list-lacks-agent
      rounds:
        - 1
      status: open
      class: consistency
      severity: NIT
      fix: mechanical
---
Round 1 is the first round.
The open weight sits on the new operator agent's session lifecycle and prompt:
carry-over from a live predecessor, no end of life, a parent directive at odds with its stencil, and a silent wait under a pause.
The rest is missing or thin test coverage and small text drift.
