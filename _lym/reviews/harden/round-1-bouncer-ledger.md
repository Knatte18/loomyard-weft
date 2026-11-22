---
round: 1
ledger:
  - key: guard-push-bypass-any-paths
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: archive-tag-plain-refs-tags-one-repo
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: unpushed-commits-no-upstream-says-none
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: push-missing-fetch-refspec-no-way-forward
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: pull-missing-fetch-refspec-reports-ok
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: killed-teardown-child-orphans-unnamed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: guard-refusal-omits-sandbox-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

## Cross-round narrative

Round 1 is the first round of the second fabric hardener campaign on the one-repository layout, and no earlier ledger exists.
The reviewer drove the layout on sandbox hubs and found seven findings.

- Guard: the sandboxed-role guard passes any `lym fabric push` that carries injected path flags without checking where they point (F1),
  and its refusal text omits the sandbox hub as a way forward (F7).
- Namespace leak: a discarded task branch's archive tag is pushed to plain `refs/tags/` because the code store reads as the identity namespace (F2, BLOCKING).
- Layout read: with the records store's fetch refspecs removed, a pair push fails with a raw error and no failed-push record (F4),
  and a pull reports success while moving nothing (F5).
- Messages: a rejected first push names "local commits: none" and advises a reset that drops one (F3).
- Crash: a `kill -9` of the teardown child leaves origin branches no reader verb names (F6).

The facts list six fix commits made after the review.
All seven keys stay `open`: no fresh reviewer has seen those fixes, and a fix counts as convergence only for a key first raised in a closing round from round 2 on.
Round 2 must re-drive each fix on a sandbox hub, run the sweep of the class F2 belongs to, and drive the departures round 1 left undone.
