---
round: 3
ledger:
  - key: relay-restart-cap-unreachable
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: await-first-read-stale-state
    rounds: [1]
    status: resolved
  - key: relay-verb-location-from-hub-dir
    rounds: [1]
    status: resolved
  - key: relay-fork-bound-unstated
    rounds: [1]
    status: resolved
  - key: outbound-read-timeout-after-handler
    rounds: [1]
    status: resolved
  - key: checkbinary-type-mismatch
    rounds: [1]
    status: resolved
  - key: creel-doc-extends-falsified
    rounds: [1, 2]
    status: resolved
  - key: cards-11-13-stale-docs
    rounds: [1]
    status: resolved
  - key: step6-lock-without-probe-grace
    rounds: [1]
    status: resolved
  - key: per-helper-text-tests
    rounds: [1]
    status: resolved
  - key: readwait-subtest-duplicate
    rounds: [1]
    status: resolved
  - key: card8-edits-card7-test-file
    rounds: [1]
    status: resolved
  - key: login-empty-token-no-row
    rounds: [1]
    status: resolved
  - key: slackgeometry-credential-dir-no-row
    rounds: [1]
    status: resolved
  - key: cwd-resolution-bound-list-missing-slackengine
    rounds: [1]
    status: resolved
  - key: setup-missing-scope-names-no-scopes
    rounds: [1]
    status: resolved
  - key: paused-error-after-post-unspecified
    rounds: [2]
    status: resolved
  - key: newroot-long-missing-slack
    rounds: [2]
    status: resolved
  - key: setpresence-beams-error-outcome-unassigned
    rounds: [3]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: beams-error-branch-no-row
    rounds: [3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: restart-wait-clamp-unreachable
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: restart-cap-edges-unassigned
    rounds: [3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: readwait-lost-confirmation-contract
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card10-verb-use-strings
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised sixteen findings, round 2 four, round 3 seven.
The restart-cap key has now been open three rounds and is still BLOCKING `design`.
Round 2 put the `StartFailures` reset at the relay's first accepted read; round 3 shows that a relay dying after its first read then respawns every minute with no cap.
The proposed fix judges the last relay by its lifetime, `LastSlackOK` minus `LastSpawnAt`, against a named constant, and moves the reset to the starter.
That is new text no fresh reviewer has seen.

The other round 3 findings are new keys and all `mechanical`: the `SetPresence` and `Hub.Beams` error outcomes in card 07 with their missing rows, the unreachable 30-minute clamp and three unassigned edges in card 09, the `ReadWait` lost-confirmation wording in card 03, and card 10's `Use` strings.
The round 2 keys `creel-doc-extends-falsified`, `paused-error-after-post-unspecified` and `newroot-long-missing-slack` did not recur and are resolved.
