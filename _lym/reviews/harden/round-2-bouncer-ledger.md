---
round: 2
ledger:
  - key: guard-push-bypass-any-paths
    rounds: [1]
    status: resolved
  - key: archive-tag-plain-refs-tags-one-repo
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: unpushed-commits-no-upstream-says-none
    rounds: [1]
    status: resolved
  - key: push-missing-fetch-refspec-no-way-forward
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: pull-missing-fetch-refspec-reports-ok
    rounds: [1]
    status: resolved
  - key: killed-teardown-child-orphans-unnamed
    rounds: [1]
    status: resolved
  - key: guard-refusal-omits-sandbox-way-forward
    rounds: [1]
    status: resolved
  - key: pending-teardown-rerun-skips-owed-archive
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: archive-tags-share-one-origin-prefix
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

## Cross-round narrative

Round 1 found seven findings on the one-repository layout; round 2, under a reviewer model that differs from round 1's, re-drove all seven fixes and found four.

- Resolved on re-drive: the push guard over injected paths and its refusal wording, the unpushed-commit listing with no upstream,
  the pull that reported success while moving nothing, and the killed teardown child, whose `status` and `prune` now name the slug.
- Reopened: the archive tag of a discarded task branch lands in plain `refs/tags/` again when the code store spells origin's URL differently from the records store.
  The round-1 fix gated the records namespace on equal URL strings.
- Reopened: with the records store's fetch refspecs removed, `lym fabric reconcile` still returns a raw `gitrepo:` error through `Bolt.pullFirst`; the push and pull sites hold.
- New: the re-run a pending-teardown record names finishes the teardown without the archive the killed child owed,
  so origin loses a branch with its unlanded commit reachable only under a local tag.
- New: code-side and records-side archive tags share one origin prefix, and each store's reader takes the other's tags.

The fixes made after round 2's review are new text no fresh reviewer has seen, so every open key stays open.
Round 3 re-drives them live, closes the residue, then covers the regions in round 2's Coverage a sandbox can reach.
Smaller observations the review recorded outside its findings (a mid-seed join's wording, a missing `--into` directory, a failed-step teardown that clears its record) were fixed in the same commits and are re-driven there too.
