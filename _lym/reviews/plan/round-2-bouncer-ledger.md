---
round: 2
ledger:
  - key: raddle-name-declared-twice
    rounds: [1]
    status: resolved
  - key: plain-promote-way-forward-not-raddle
    rounds: [1]
    status: resolved
  - key: project-type-label-unsourced
    rounds: [1]
    status: resolved
  - key: unknown-decision-pattern-id-unchecked
    rounds: [1]
    status: resolved
  - key: store-slug-bound-overstates-start-guard
    rounds: [1]
    status: resolved
  - key: raddle-stencil-omits-parent-edit-directives
    rounds: [1]
    status: resolved
  - key: arbiter-recipe-description-raddle
    rounds: [1]
    status: resolved
  - key: filing-level-departs-from-parent-constraint
    rounds: [1]
    status: resolved
  - key: filed-task-body-unrendered
    rounds: [1]
    status: resolved
  - key: loomcli-doc-go-left-stale
    rounds: [1]
    status: resolved
  - key: status-draft-comment-false
    rounds: [1]
    status: resolved
  - key: docs-overview-misses-config-keys
    rounds: [1]
    status: resolved
  - key: legacy-sketch-label-tasks-unwritable
    rounds: [1]
    status: resolved
  - key: discussed-promotion-replay-bypass-unbounded
    rounds: [1]
    status: resolved
  - key: batten-guard-pass-branch-rationale
    rounds: [1]
    status: resolved
  - key: handoff-post-result-dropped
    rounds: [1]
    status: resolved
  - key: slug-sketch-never-fires-alone
    rounds: [1]
    status: resolved
  - key: card-4-research-steps
    rounds: [1]
    status: resolved
  - key: session-end-nil-hook-row-redundant
    rounds: [1]
    status: resolved
  - key: amendment-target-comment-missed
    rounds: [1]
    status: resolved
  - key: loomshed-allowlist-not-extended
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: pawlpaths-handoff-marker-untargeted
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: raddle-table-return-note-not-optional
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: parse-strictness-no-tolerant-mechanism
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: shedcli-recipe-key-literal
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: splicepart-halt-unreachable
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: part-label-sources-unstated
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: stencil-no-question-rule-scan-wording
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised twenty findings, and the fresh round 2 review raised none of those keys again, so all twenty are marked resolved.
Round 2 raised eight new keys, F1 to F8 in review order, none BLOCKING.
The only gating-class one is `splicepart-halt-unreachable`, which is first raised in round 2 and is carried into the decision record's Open risks.
All eight fixes are local corrections whose result the finding text already fixes.
No relabel was applied.
