---
round: 2
ledger:
  - key: slug-naming-prime-push-replays-takeover
    rounds: [1]
    status: resolved
  - key: teardown-deps-seams-break-reflective-tests
    rounds: [1, 2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: dobby-pusher-for-pair-owned-elsewhere
    rounds: [1]
    status: resolved
  - key: pusher-verb-resolve-refusal-no-way-forward
    rounds: [1]
    status: resolved
  - key: teardown-loops-on-ended-deleted-branch
    rounds: [1]
    status: resolved
  - key: prime-records-archive-failure-unreached
    rounds: [1]
    status: resolved
  - key: tender-resume-help-and-pushdeferred-doc-stale
    rounds: [1]
    status: resolved
  - key: owed-unrecorded-test-duplicates-existing-rows
    rounds: [1]
    status: resolved
  - key: restartpusher-test-retests-episode-logging
    rounds: [1]
    status: resolved
  - key: restartpusher-misplaced-in-battencli
    rounds: [1]
    status: resolved
  - key: statuspushlockwait-doc-stale
    rounds: [1]
    status: resolved
  - key: pusher-started-has-no-source-value
    rounds: [1]
    status: resolved
  - key: ended-records-branch-restarts-pusher-every-tick
    rounds: [1]
    status: resolved
  - key: beamcli-doc-omits-pusher-restart
    rounds: [1]
    status: resolved
  - key: pushowed-test-header-names-transition-seam
    rounds: [1]
    status: resolved
  - key: tracking-ref-delete-unsets-upstream
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1 raised 15 findings, 4 BLOCKING.
Round 2 resolved 14 of them, leaving the seam-list contradiction in card 10 open.

The two open keys are the two BLOCKING findings of round 2.
`teardown-deps-seams-break-reflective-tests` is the reopened round 1 key: card 10 now edits both reflective seam lists, but the lists exclude each other.
`tracking-ref-delete-unsets-upstream` is the gap behind the accepted ended-branch teardown fix: the only existing delete also unsets the branch's upstream, and no card targets `gitrepo`.
