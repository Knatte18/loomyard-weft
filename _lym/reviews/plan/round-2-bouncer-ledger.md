---
round: 2
ledger:
  - key: relay-restart-cap-unreachable
    rounds: [1, 2]
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
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
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
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: newroot-long-missing-slack
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised sixteen findings; round 2 raised four.
Only the restart-cap and creel-doc topics came back, as F1 and F2.
F1 is the BLOCKING `design` finding: the starter's write order on a failing spawn and the owner of the `StartFailures` reset are unassigned in card 09.
F2 is the `internal/creelcli/doc.go` Extends-versus-Edit mismatch that `ReadWait` causes.
F3 (`Hub.Paused` error after a successful post, card 07) and F4 (`newRoot` `Long` list, card 10) are new.
The other round 1 keys did not recur and are resolved.
