---
round: 2
ledger:
  - key: bolt-lock-path-not-refused
    rounds: [1]
    status: resolved
  - key: headroom-commit-dropped-unrelated-pattern-text
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: shedadapters-lock-move-untested
    rounds: [1]
    status: resolved
  - key: refuse-tracked-tree-skips-after-earlier-lym-segment
    rounds: [1]
    status: resolved
  - key: stale-push-lock-residue-comment
    rounds: [1]
    status: resolved
  - key: moved-lock-behaviours-lack-rows
    rounds: [1]
    status: resolved
---

Round 1 raised six findings.
Round 2 raised one, F1, which is the remaining part of the headroom finding: the durable-vs-ephemeral entry still lacks "rendered" and the "temporary directory" wording in its heddle clause.
The other five round 1 keys are not raised again, and the round 2 review's settled list covers their subjects, so they are resolved.
