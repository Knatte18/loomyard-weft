---
round: 4
ledger:
  - key: guard-push-bypass-any-paths
    rounds: [1]
    status: resolved
  - key: archive-tag-plain-refs-tags-one-repo
    rounds: [1, 2]
    status: resolved
  - key: unpushed-commits-no-upstream-says-none
    rounds: [1]
    status: resolved
  - key: push-missing-fetch-refspec-no-way-forward
    rounds: [1, 2]
    status: resolved
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
    status: resolved
  - key: archive-tags-share-one-origin-prefix
    rounds: [2]
    status: resolved
  - key: killed-clone-leaves-hub-without-way-forward
    rounds: [4]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: merge-skips-records-layout-read
    rounds: [4]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: drop-local-archive-tags-pushed-by-remote-teardown
    rounds: [4]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: plain-archive-ref-named-by-no-verb-one-repo
    rounds: [4]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: discard-rerun-way-forward-names-no-command
    rounds: [4]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

## Cross-round narrative

Round 1 found seven findings on the one-repository layout.
Round 2 re-drove the fixes and found four, three BLOCKING.
Rounds 3 and 4 raised no frontmatter finding, and round 4 re-drove the earlier fixes live, so all nine earlier keys stay resolved.

Round 4 was incremental and could record new defects only as prose observations O1 to O5; they are carried here as open.
`killed-clone-leaves-hub-without-way-forward` (O1) is the only MEDIUM.
It is first raised in round 4, so it is an open risk and not a block.
`ly:orch` filed all five on the board as note `harden-fabric-undriven-cases` for a follow-up task.

Undriven paths carried with them: the owner-archive push, the hand-over and code-tip reset paths, and a losing lease.
