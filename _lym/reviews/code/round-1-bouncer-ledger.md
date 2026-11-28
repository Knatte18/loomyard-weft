---
round: 1
ledger:
    - key: batch-gate-key-not-one-constant
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: mechanical
    - key: narrowed-tier-main-packages-untested
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: whole-running-tier-treated-as-narrowed
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: narrowed-tier-watch-skip-untested
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: plan-spec-batch-scope-contradiction
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: card-flag-help-old-scope
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: summary-names-decision-record
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
---
Round 1 opened every key above.
The gating ones are the batch gate key's name, spelled in more than one place,
and the missing coverage of the main packages in a narrowed batch tier.
The rest concern a whole-running tier treated as narrowed, the narrowed tier's watch skip coverage,
and prose in the plan spec, the flag help and the change description.
