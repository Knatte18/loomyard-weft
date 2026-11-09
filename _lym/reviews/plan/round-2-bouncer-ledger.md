---
round: 2
ledger:
  - key: redundant-planned-tests
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: outcome-part-merge-unspecified
    rounds: [1]
    status: resolved
  - key: follow-never-ends-on-rewritten-record
    rounds: [1]
    status: resolved
  - key: worker-follow-verdict-reachable
    rounds: [1]
    status: resolved
  - key: adopt-long-text-over-cap
    rounds: [1]
    status: resolved
  - key: stale-falsified-text-set
    rounds: [1, 2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: integration-test-forced-change-undescribed
    rounds: [1]
    status: resolved
  - key: dead-outputs-and-constants
    rounds: [1]
    status: resolved
  - key: fabricengine-scratch-docs-stale
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: standing-move-refusal-unplaced
    rounds: [1]
    status: resolved
  - key: card2-test-sentence-wrong-recorder
    rounds: [1]
    status: resolved
  - key: reconcile-warnings-wrong-step
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: failing-step-part-fate-unstated
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: seeding-from-subjects-adds-root-item
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: shared-decisions-overstate-rerun-text
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's BLOCKING test-economy finding and its gating MEDIUM design findings no longer appear in round 2; their keys are resolved except where noted.
Test economy recurs at a lower level as F4 and F5 (`redundant-planned-tests`, consistency MEDIUM).
`stale-falsified-text-set` recurs as F6 (comments no card rewords), and `fabricengine-scratch-docs-stale` as F7 (`lymdirs.AdoptDirName`).
Round 2 adds four keys: the reconcile warnings assigned to a step that cannot compute them (F1, design MEDIUM), the unstated fate of a failing step's part (F2, decision MEDIUM), the root item seeded into the slugs list (F3) and the overstated re-run text in Shared Decisions (F8).
