---
round: 2
ledger:
  - key: required-to-on-rewrite-drops-meaning
    rounds: [1]
    status: resolved
  - key: per-task-keys-declared-at-several-sites
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: reinterpret-reads-outside-the-one-lookup
    rounds: [1]
    status: resolved
  - key: task-setting-chain-wrong-lymcwd-api
    rounds: [1]
    status: resolved
  - key: carry-misses-runs-in-flight
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: worktree-files-left-out-of-reinterpretation
    rounds: [1]
    status: resolved
  - key: bad-beam-run-settings-pass-upsert
    rounds: [1]
    status: resolved
  - key: unread-land-notice-dropped-silently
    rounds: [1]
    status: resolved
  - key: carried-legacy-off-overrides-parent
    rounds: [1]
    status: resolved
  - key: replay-does-not-fold-legacy-keys
    rounds: [1]
    status: resolved
  - key: reinterpretation-registration-untested
    rounds: [1]
    status: resolved
  - key: member-line-overrides-explicit-off
    rounds: [1, 2]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: beam-project-notify-on-land-dropped
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: advisory-read-aborts-reconcile
    rounds: [1]
    status: resolved
  - key: old-clear-owed-memory-second-notice
    rounds: [1]
    status: resolved
  - key: board-skill-prune-rule-stale
    rounds: [1]
    status: resolved
  - key: overview-resolver-input-stale
    rounds: [1]
    status: resolved
  - key: strictness-section-disagrees-with-layers-rule
    rounds: [1]
    status: resolved
  - key: chain-builder-contracts-untested
    rounds: [1]
    status: resolved
  - key: merge-demand-clause-unasserted
    rounds: [1]
    status: resolved
  - key: contained-tests-lack-keep-markers
    rounds: [1]
    status: resolved
  - key: halt-way-forward-wrong-in-two-cases
    rounds: [1]
    status: resolved
  - key: lookup-skips-envsource-expansion
    rounds: [1]
    status: resolved
  - key: blast-radius-too-narrow
    rounds: [1]
    status: resolved
  - key: seam-names-still-say-parent
    rounds: [1]
    status: resolved
  - key: reinterpret-discovers-worktrees-and-anchors
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: registration-test-lacks-keep-marker
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: duplicate-row-in-prepare-run-settings
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
---

Round 1 raised 25 findings; round 2 closes 21 of those keys and carries four forward.

Carried open: the beam module name still written as a literal at three sites (F2, mechanical), the carry into seeded runs that misses other machines and an early board save (F5), the project's `notify_on_land` lost by the fold before the carry (F13), and the member-line exception (F12, a `decision` finding deferred to the parent).

New in round 2: `settingPlaces` discovers worktrees and anchors itself instead of taking told geometry (F26, BLOCKING, design-wrong), a missing `//testtiming:keep` marker on the registration test (F27), and a duplicate row in `TestPrepareRunSettings` (F28).
The two test findings are mechanical.
