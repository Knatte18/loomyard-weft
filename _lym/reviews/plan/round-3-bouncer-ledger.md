---
round: 3
ledger:
  - key: slug-naming-prime-push-replays-takeover
    rounds: [1]
    status: resolved
  - key: teardown-deps-seams-break-reflective-tests
    rounds: [1, 2]
    status: resolved
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
    status: resolved
---

Round 1 raised 15 findings, 4 BLOCKING; round 2 resolved 14 and left two BLOCKING keys.
Round 3 found nothing and resolved both: card 10 now places the teardown seams only in `nilLegal`, and card 1 adds `gitrepo.Repo.DeleteRemoteTrackingRef`, which keeps the branch's upstream config.
Every key is resolved.
