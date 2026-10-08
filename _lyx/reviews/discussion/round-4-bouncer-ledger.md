---
round: 4
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
    status: open
    class: design
    severity: MEDIUM
  - key: package-level-false-positive-bound-understated
    rounds: [4]
    status: open
    class: design
    severity: MEDIUM
  - key: plan-writer-note-misplaces-delete-before-reference-site
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
  - key: scope-d6-file-paths-qualifier-mismatch
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
  - key: no-criterion-head-answers-outside-canonicalization
    rounds: [4]
    status: open
    class: scope
    severity: LOW
  - key: rename-to-side-criterion-direction-unnamed
    rounds: [4]
    status: open
    class: scope
    severity: LOW
  - key: d5-unit-taken-from-glyph-unitpath
    rounds: [4]
    status: open
    class: consistency
    severity: NIT
---

Rounds 1 and 2 raised the older themes (verdict identity, the re-sign arrow head, restatements outside the support log); they were resolved by round 3 and did not return.
Round 3's three open keys are resolved: D5 now joins the head match to `CanonicalizeHandles`, which the chokepoint pin accepts, and Scope and the Rename to-side criterion were reworded.
Round 4 raises seven new keys.
Two gate: caller coverage by an earlier card is unsound for a Rename Old side or re-signed member (F1), and the over-reporting bound understates the package-level textual match (F2).
The other five are LOW or NIT wording and criterion gaps: the call-site note (F3), the file-paths qualifier (F4), head answers staying out of canonicalization (F5), the Rename to-side direction (F6), and the `Glyph.UnitPath` wording (F7).
