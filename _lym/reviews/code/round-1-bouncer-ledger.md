---
round: 1
ledger:
  - key: fetch-prune-deletes-local-tags
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: merge-danger-understated
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: norpair-rejudge-reads-unfetched-code-ref
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: no-row-for-failed-second-read
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: pair-absent-refusal-without-machine-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first review, so no earlier ledger carries forward.
The one gating finding is that `Repo.Fetch`'s bare `--prune` also prunes `refs/tags/*` in a one-repository records store;
its fix changes the approach, so it needs a fresh review.
The merge-danger rewrite waits on that fix.
The other three findings are LOW test and wiring corrections whose text fixes the result.
