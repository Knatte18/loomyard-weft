---
round: 2
ledger:
  - key: round-numbering-vs-burler-rounds
    rounds: [1]
    status: resolved
  - key: bouncer-nowhere-entries-placement
    rounds: [1]
    status: resolved
  - key: d11-migration-without-discussion-producer
    rounds: [1]
    status: resolved
  - key: advisors-startable-from-local-layers-unbounded
    rounds: [1, 2]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: template-defaults-undecided
    rounds: [1]
    status: resolved
  - key: numeric-seat-name-collides-with-lead
    rounds: [1]
    status: resolved
  - key: goal2-contradicts-migrated-names
    rounds: [1]
    status: resolved
  - key: arbiter-reader-cannot-draw-unreadable-attempt
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: attempt-without-verdict-reads-running-forever
    rounds: [1, 2]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d5-pause-record-retention-bound
    rounds: [1]
    status: resolved
  - key: d8-guard-bound-wrong-direction
    rounds: [1]
    status: resolved
  - key: d11-advisors-migration-cross-layer
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: goto-entry-has-no-round
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: hand-on-can-open-round
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: continue-not-arbiter-verdict
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's two BLOCKING keys, the round-numbering premise and D11's migration over documents without `discussion_producer`, were not raised again and are resolved, along with the other round-1 keys round 2 did not re-raise.
Three round-1 keys continue in narrower form: advisor reach (F2), the unreadable-attempt placement (F4) and the attempt verdict source (F3).
Round 2 adds four new keys: the cross-layer advisors migration (F1, design MEDIUM, carried to Open risks), a goto entry with no round (F5), a hand-on that can open a round (F6) and the `continue` verdict word (F7).
