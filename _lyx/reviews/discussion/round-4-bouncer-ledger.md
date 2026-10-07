---
round: 4
ledger:
  - key: d5-marker-path-stale-or-unreachable
    rounds: [1, 2]
    status: resolved
  - key: d6-resume-partial-or-mixed-round
    rounds: [1, 2, 3, 4]
    status: open
    class: design
    severity: MEDIUM
  - key: d1-d6-d7-join-gate-ordering-single-gate
    rounds: [1, 2]
    status: resolved
  - key: d8-disputed-criterion-presumes-checker
    rounds: [1, 2]
    status: resolved
  - key: d9-resolution-at-config-load
    rounds: [1]
    status: resolved
  - key: decisions-without-acceptance-criterion
    rounds: [1]
    status: resolved
  - key: fixer-deadline-vs-waiting-time
    rounds: [1]
    status: resolved
  - key: auto-mode-assumptions-restate-rationale
    rounds: [1]
    status: resolved
  - key: d1-no-shuttle-capability-vs-stop
    rounds: [2, 3]
    status: resolved
  - key: marker-standalone-path-and-cleanup
    rounds: [2, 3, 4]
    status: open
    class: consistency
    severity: LOW
  - key: fixer-may-edit-review-file
    rounds: [2, 3]
    status: resolved
  - key: d1-first-terminal-event-wording
    rounds: [2]
    status: resolved
  - key: fixer-done-before-marker
    rounds: [3]
    status: resolved
  - key: d5-bound-overstates-prompt-only-release
    rounds: [3, 4]
    status: open
    class: design
    severity: LOW
  - key: d6-failed-removal-of-stopped-half
    rounds: [4]
    status: open
    class: decision
    severity: MEDIUM
  - key: d6-resume-restarts-live-fixer
    rounds: [4]
    status: open
    class: decision
    severity: LOW
  - key: gate-pass-on-cap-restated
    rounds: [4]
    status: open
    class: consistency
    severity: NIT
---

Round 4 maps its findings as follows:

- F1 is `d6-resume-partial-or-mixed-round`, open since round 1: D6's resume branch is written for the happy path, and its failure branches, the tamper comparison and the neither-live case are unspecified.
  It is the only gating finding, and it recurs, hence `CIRCLING`.
- F2 is new: `d6-failed-removal-of-stopped-half`.
- F3 is new: `d6-resume-restarts-live-fixer`.
- F4 is `d5-bound-overstates-prompt-only-release`, still open, now naming the second early-release route.
- F5 is `marker-standalone-path-and-cleanup`, back at LOW as D5 and the Constraints state the location differently.
- F6 is new: `gate-pass-on-cap-restated`.

Resolved this round: `d1-no-shuttle-capability-vs-stop` (round 4 confirms the strand guid, the remover and the shared review-ready function are named), `fixer-done-before-marker` (settled for a fresh round) and `fixer-may-edit-review-file` (D7's byte comparison bounds a fresh round; the resume remainder is carried in F1).
