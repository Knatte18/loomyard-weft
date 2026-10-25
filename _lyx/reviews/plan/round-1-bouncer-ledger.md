---
round: 1
ledger:
  - key: detached-pass-hub-root-cwd
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: forced-edits-outside-cards
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: innerrun-state-change-lost-between-passes
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: batten-arbiter-goto-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: dobby-state-undecodable-no-way-forward
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: records-stop-line-unlicensed
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: card23-resign-arrows-missing
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: spawn-hold-owner-recheck-under-pause
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: land-pause-reaches-arbiter
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: card34-bolt-pull-unlicensed
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: hidden-dobby-verb-no-bound
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: batten-landing-halt-row-unreachable
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pass-exit-code-prerun-claim-unreal
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: parent-dobby-spawned-where-foreign-owned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: cancel-restart-refusals-meet-draft
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: adopt-named-call-live-pair
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: failed-revive-text-no-next-step
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: tidy-timeout-loses-landings
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: stale-docs-and-pattern-backgrounds
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: tidy-spec-segment-skills
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: ensurelive-prime-claim-failure
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
---

Round 1 is the first round; no earlier ledger exists.
The two BLOCKING findings are structural.
Detached passes (cards 33 and 28) start from the hub root, where `lyxcwd.Resolve` refuses, so no pass can arm.
The plan also omits tests, callers and import allowlists that cards 13, 28, 35 and 36 force outside their own packages.
Three more `design` findings at MEDIUM concern the InnerRun state-change observation, the batten arbiter goto bound, and an undecodable dobby state record.
The remaining findings are bounds, texts, glyph arrows and doc staleness, mostly mechanical.
