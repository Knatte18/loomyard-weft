---
round: 2
ledger:
  - key: detached-pass-hub-root-cwd
    rounds: [1]
    status: resolved
  - key: forced-edits-outside-cards
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: innerrun-state-change-lost-between-passes
    rounds: [1]
    status: resolved
  - key: batten-arbiter-goto-unbounded
    rounds: [1]
    status: resolved
  - key: dobby-state-undecodable-no-way-forward
    rounds: [1]
    status: resolved
  - key: records-stop-line-unlicensed
    rounds: [1]
    status: resolved
  - key: card23-resign-arrows-missing
    rounds: [1]
    status: resolved
  - key: spawn-hold-owner-recheck-under-pause
    rounds: [1]
    status: resolved
  - key: land-pause-reaches-arbiter
    rounds: [1]
    status: resolved
  - key: card34-bolt-pull-unlicensed
    rounds: [1]
    status: resolved
  - key: hidden-dobby-verb-no-bound
    rounds: [1]
    status: resolved
  - key: batten-landing-halt-row-unreachable
    rounds: [1]
    status: resolved
  - key: pass-exit-code-prerun-claim-unreal
    rounds: [1]
    status: resolved
  - key: parent-dobby-spawned-where-foreign-owned
    rounds: [1]
    status: resolved
  - key: cancel-restart-refusals-meet-draft
    rounds: [1]
    status: resolved
  - key: adopt-named-call-live-pair
    rounds: [1]
    status: resolved
  - key: failed-revive-text-no-next-step
    rounds: [1]
    status: resolved
  - key: tidy-timeout-loses-landings
    rounds: [1]
    status: resolved
  - key: stale-docs-and-pattern-backgrounds
    rounds: [1]
    status: resolved
  - key: tidy-spec-segment-skills
    rounds: [1]
    status: resolved
  - key: ensurelive-prime-claim-failure
    rounds: [1]
    status: resolved
  - key: innerrun-poll-seam-dead
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: passexit-type-uncreated
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised 21 findings; the round 2 re-check found every fix present except for the forced-edits gap.
`forced-edits-outside-cards` recurs at MEDIUM: more tests, import allowlists and a recipe header comment in cards 4, 7, 13, 17, 27 and 36 break with no card listing them.
Two new LOW consistency findings: `InnerRun`'s Sleep seam and poll cadence survive card 13 with no reader, and card 33's `PassExit` type has no Create entry.
