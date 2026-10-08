---
round: 5
ledger:
  - key: plan-gate-only-bound-not-implementable-on-shared-seams
    rounds: [1]
    status: resolved
  - key: d1-first-card-depends-on-later-checks
    rounds: [1]
    status: resolved
  - key: no-mechanism-turns-exposed-defect-into-card
    rounds: [1]
    status: resolved
  - key: d5-reference-scan-limits-unstated
    rounds: [1]
    status: resolved
  - key: resign-arrow-head-unbounded
    rounds: [1, 2]
    status: resolved
  - key: no-double-report-rule-delete-only
    rounds: [1]
    status: resolved
  - key: goal-overclaims-verdict-identity
    rounds: [1, 2]
    status: resolved
  - key: d2-bound-names-wrong-sequencing-unit
    rounds: [1]
    status: resolved
  - key: restatements-outside-support-log
    rounds: [1, 2]
    status: resolved
  - key: open-risks-promises-unbuilt-rubric-backstop
    rounds: [2]
    status: resolved
  - key: d3-stability-names-wrong-transition
    rounds: [2]
    status: resolved
  - key: d4-d6-both-claim-stencil-edit
    rounds: [2]
    status: resolved
  - key: d5-quarry-name-breaks-pinned-chokepoint
    rounds: [3]
    status: resolved
  - key: scope-omits-second-rubric-finding
    rounds: [3]
    status: resolved
  - key: no-criterion-rename-to-side-handle-redundancy
    rounds: [3]
    status: resolved
  - key: caller-coverage-earlier-card-unsound-for-rename-resign
    rounds: [4]
    status: resolved
  - key: package-level-false-positive-bound-understated
    rounds: [4]
    status: resolved
  - key: plan-writer-note-misplaces-delete-before-reference-site
    rounds: [4]
    status: resolved
  - key: scope-d6-file-paths-qualifier-mismatch
    rounds: [4]
    status: resolved
  - key: no-criterion-head-answers-outside-canonicalization
    rounds: [4]
    status: resolved
  - key: rename-to-side-criterion-direction-unnamed
    rounds: [4]
    status: resolved
  - key: d5-unit-taken-from-glyph-unitpath
    rounds: [4]
    status: resolved
  - key: rename-caller-coverage-contradicts-spec-rename-mechanic
    rounds: [5]
    status: open
    class: design
    severity: MEDIUM
  - key: d5-clearing-remedy-collides-with-d3-redundancy
    rounds: [5]
    status: open
    class: consistency
    severity: MEDIUM
  - key: file-and-package-delete-outside-caller-coverage
    rounds: [5]
    status: open
    class: design
    severity: MEDIUM
  - key: resign-arrow-unwritable-for-interface-methods-and-fields
    rounds: [5]
    status: open
    class: decision
    severity: MEDIUM
  - key: own-package-undefined-for-external-test-packages
    rounds: [5]
    status: open
    class: decision
    severity: MEDIUM
  - key: two-card-resign-caller-coverage-unpinned
    rounds: [5]
    status: open
    class: decision
    severity: LOW
  - key: changed-files-guard-cited-as-bound-but-informational
    rounds: [5]
    status: open
    class: consistency
    severity: LOW
---

Rounds 1 to 3 raised and closed the older themes: verdict identity, the re-sign arrow head, restatements outside the support log, the chokepoint pin on `quarry.Name`, and the Scope and Rename to-side wording.
Round 4's seven keys were not raised again in round 5 and are marked resolved; the same-card caller rule replaced the earlier-card rule.
Round 5 raises seven new keys, mostly around D5's caller-coverage check.
Two gate: the same-card Rename rule conflicts with the spec's Rename mechanic (F1), and file and package `Delete` targets bypass the check (F3).
The rest are D5/D3 interaction (F2), member kinds that cannot take the re-sign arrow (F4), the undefined "own package" for external test packages (F5), the two-card re-sign criterion (F6), and the informational changed-files guard cited as a bound (F7).
