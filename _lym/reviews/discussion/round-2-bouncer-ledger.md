---
round: 2
ledger:
  - key: d1-add-inputs-refuse-live-pair
    rounds: [1]
    status: resolved
  - key: d2-land-way-forward-does-not-recover
    rounds: [1]
    status: resolved
  - key: d4-push-departs-push-both-sides
    rounds: [1]
    status: resolved
  - key: d1-reconcile-change-underspecified
    rounds: [1]
    status: resolved
  - key: d2-permanent-transient-split-by-exclusion
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-cancel-at-land-misreads-childreachedparent
    rounds: [1]
    status: resolved
  - key: d3-cancel-lifts-owner-pause
    rounds: [1]
    status: resolved
  - key: d6-lease-tip-disagrees-with-d4-push
    rounds: [1]
    status: resolved
  - key: d6-sweep-has-no-origin-tip-archive
    rounds: [1]
    status: resolved
  - key: open-risks-defer-research
    rounds: [1]
    status: resolved
  - key: d1-bound-misses-surviving-local-weft
    rounds: [1]
    status: resolved
  - key: d4-diverged-tip-not-fetched
    rounds: [1]
    status: resolved
  - key: d5-rule-reads-only-remote-tip
    rounds: [1]
    status: resolved
  - key: d7-bound-silent-on-longer-hold
    rounds: [1]
    status: resolved
  - key: d7-criterion-pins-momentary-state
    rounds: [1]
    status: resolved
  - key: d1-adoptlocal-rule-unnamed
    rounds: [1]
    status: resolved
  - key: d8-drop-leaves-upstream-config
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d6-bound-rests-on-lock-sync-push-skips
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d8-cause-asserted-not-shown
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d8-local-warp-stale-tracking-ref
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d2-before-land-cancel-keeps-pair-parts
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: d1-task-worktree-alone-gone-failed
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d2-newest-archive-tag-order-undefined
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d2-sweep-undoes-land-restore
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d1-absent-pair-origin-record-unreadable
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-owned-elsewhere-reason-false
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: constraints-timeline-is-narrative
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d6-archives-exactly-lease-tip-wording
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: d1-prune-not-slug-scoped
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: open-risks-narrates-history-and-repeats
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1's sixteen keys were not re-raised by round 2's review, except the transient-bucket key, so they are marked resolved; the round-2 focus verdicts also confirm D4 now complies with the declared `PATTERN-push-both-sides` text.
`d2-permanent-transient-split-by-exclusion` recurs as F2: the closed set of permanent refusals still misses fork-point checks that guard nothing under adopt-only, the checked-out-elsewhere adopt-local refusal and a stray path at the task worktree.

Round 2 adds D8's gaps.
The BLOCKING F1: dropping the tracking ref leaves the branch's upstream config, so `UpstreamSHA` errors and every push of the re-adopted weft refuses.
Related MEDIUM findings: the 2026-11-03 cause is unproven (F4) and an adopted local warp branch keeps its stale tracking ref (F5).
Other MEDIUM findings: D6's bound ignores the synchronous commit-records push (F3), the way forward before Land cancels without `--remove` (F6), and a run whose task worktree alone is gone is failed (F7).
The rest are LOW and NIT wording or relocation fixes; F6 and the consistency findings are marked mechanical because the review text fixes their result.
