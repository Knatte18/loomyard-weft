---
round: 1
ledger:
  - key: comment-line-breaks-semicolon-and-midphrase
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: orch-tests-missing-t-parallel
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: empty-reload-retry-uncovered
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: orch-warn-vocabulary-unasserted
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: reload-step-literal-zero
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 is the first review, so every entry opens here.
Two consistency findings concern comment line breaks and missing `t.Parallel`.
Two scope findings concern test coverage gaps in `orchengine`: the empty `reload_retry` branch and the skill-skip warn vocabulary.
One NIT concerns resetting `ReloadStep` to a literal instead of `ReloadStepSkills`.
