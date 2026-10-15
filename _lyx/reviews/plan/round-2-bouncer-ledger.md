---
round: 2
ledger:
  - key: work-contains-out-shadowed-by-overlap
    rounds: [1]
    status: resolved
  - key: pack-check-code-row-hits-records-refusal-first
    rounds: [1]
    status: resolved
  - key: default-out-fails-without-scratch-dir
    rounds: [1]
    status: resolved
  - key: single-spelling-rule-too-broad
    rounds: [1]
    status: resolved
  - key: report-header-omits-records-and-code-paths
    rounds: [1]
    status: resolved
  - key: marker-row-obligation-uncarried
    rounds: [1]
    status: resolved
  - key: filters-spell-directory-roots-outside-variable-block
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: appended-records-path-breaks-identical-report-assertion
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: empty-arm-naming-rule-excludes-unknown
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: readonly-chmod-rows-lack-cleanup-restore
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: header-comment-update-rule-too-broad
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1's six findings are not raised again in round 2 and are resolved.
Round 2 raised five new findings (F1 to F5), mapped in order to the open keys above.
F1 is BLOCKING: the cards have the jq filters need directory roots that card 1's variable block does not hold.
The rest are local consistency or test-hygiene corrections the findings already specify; none is `design` class.
