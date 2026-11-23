---
round: 1
ledger:
  - key: slug-naming-prime-push-replays-takeover
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: teardown-deps-seams-break-reflective-tests
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: dobby-pusher-for-pair-owned-elsewhere
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: pusher-verb-resolve-refusal-no-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: teardown-loops-on-ended-deleted-branch
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: prime-records-archive-failure-unreached
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: tender-resume-help-and-pushdeferred-doc-stale
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: owed-unrecorded-test-duplicates-existing-rows
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: restartpusher-test-retests-episode-logging
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: restartpusher-misplaced-in-battencli
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: statuspushlockwait-doc-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pusher-started-has-no-source-value
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
  - key: ended-records-branch-restarts-pusher-every-tick
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: beamcli-doc-omits-pusher-restart
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: pushowed-test-header-names-transition-seam
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised 15 findings: 4 BLOCKING, 5 MEDIUM, 4 LOW, 2 NIT.

The BLOCKING findings are a prime push that still replays a taken-over batten run, the card 10 seams missing from the reflective seam tests, the dobby starting a pusher for a pair another machine owns, and the pusher verb's refusal with no way forward.
The design-class MEDIUMs are the ended-branch teardown loop, the unreached prime-records archive failure, and the two duplicated tests (card 1's owed-push test, card 8's `TestRestartPusher`).
No key has recurred, since this is the first round.
