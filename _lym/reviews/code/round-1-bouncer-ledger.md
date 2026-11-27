---
round: 1
ledger:
  - key: nested-fork-under-self-check-fork-escapes-target-check
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: session-spawned-fork-inherited-exclusion-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: missing-spawner-transcript-error-matches-errnotexist
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: self-check-stencil-says-only-fixer-spawns
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: merge-danger-blast-radius-names-only-burler
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: forkaudit-doc-understates-forks-contents
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised the keys above, all new.

The gating one concerns `auditFixRound` in `internal/burlerengine/cluster.go`:
it leaves out every report marked `SpawnedBySubagent`, whatever spawned it.
Its fix makes the exemption depend on the spawner's kind and adds the spawner's transcript path to `ForkReport`,
so it changes an interface and needs a fresh review.

The rest concern a missing test, the identity of a new error, and wording in a stencil, a doc comment
and the change description.
