---
round: 3
ledger:
  - key: beam-run-hold-blocks-membership
    rounds: [1]
    status: resolved
  - key: pawl-runs-fill-dobby-max-runs
    rounds: [1]
    status: resolved
  - key: landing-hold-defined-three-ways
    rounds: [1]
    status: resolved
  - key: fork-window-after-land-recheck
    rounds: [1]
    status: resolved
  - key: adopt-root-before-beam-strands-run
    rounds: [1]
    status: resolved
  - key: adopt-starts-beam-parts-mid-landing
    rounds: [1]
    status: resolved
  - key: ways-forward-name-shortcut
    rounds: [1]
    status: resolved
  - key: ensurelive-callers-restart-beam-dobby
    rounds: [1]
    status: resolved
  - key: runproc-role-scoped-end-deferred
    rounds: [1]
    status: resolved
  - key: resume-owner-claim-left-in-place
    rounds: [1]
    status: resolved
  - key: watcher-owner-read-failure-direction
    rounds: [1]
    status: resolved
  - key: can-land-trigger-and-episode-memory
    rounds: [1]
    status: resolved
  - key: beam-await-crash-leaves-beam-stuck
    rounds: [1]
    status: resolved
  - key: no-root-owner-sentence-contradiction
    rounds: [1]
    status: resolved
  - key: d3-before-writes-contradicts-d1
    rounds: [1]
    status: resolved
  - key: entry-no-adopt-of-beam-unread
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-gaps
    rounds: [1]
    status: resolved
  - key: d11-carries-deliberation
    rounds: [1]
    status: resolved
  - key: beamlandrefusal-orch-blocker-pawl-callers
    rounds: [2]
    status: resolved
  - key: accept-retry-leaves-dobby-down
    rounds: [2]
    status: resolved
  - key: accept-stop-before-hold-write-window
    rounds: [2]
    status: resolved
  - key: accept-recheck-refuses-own-accept
    rounds: [2]
    status: resolved
  - key: beam-await-driver-respawn-cap-silent
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: run-lock-lift-unbounded
    rounds: [2]
    status: resolved
  - key: mixed-version-bound-false-for-recipe-pawl
    rounds: [2, 3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: adopt-root-alone-leaves-beam-undriven
    rounds: [2]
    status: resolved
  - key: cancel-refusal-reads-stale-bolt
    rounds: [2]
    status: resolved
  - key: anchor-lost-on-rerun-after-died-first-pass
    rounds: [2]
    status: resolved
  - key: finalize-backstop-names-refused-way-forward
    rounds: [2]
    status: resolved
  - key: d1-ensurelive-caller-list-incomplete
    rounds: [2]
    status: resolved
  - key: criterion-tests-case-that-cannot-arise
    rounds: [2]
    status: resolved
  - key: accept-dobby-restart-failure-unbounded
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
---

Rounds 1 and 2 are compressed here: round 1's eighteen keys and eleven of round 2's thirteen are resolved, since round 3's fresh review does not raise them.
Round 3 raised three findings and no BLOCKING one.

The D9 driver-notice key is the one that recurs.
Round 2 asked for a bound on the Beam-Await driver's respawns.
The fix suppressed the driver-dead notice below `respawnCap`, and round 3 finds that suppression rests on a false premise about `noticeCondition` and that a driver which writes its status and dies again resets the count and is never reported.
The next fix should drop the suppression and keep only the quiet-notice suppression, or state a bound the status write cannot reset.

The failed-restart gap on D9's failed-accept path is new: the record says what follows a failed re-check but not a restart that errors.
D5's older-build bound is a local overstatement: only the `normalizeTask` paths refuse a beam project entry carrying recipe pawl.
