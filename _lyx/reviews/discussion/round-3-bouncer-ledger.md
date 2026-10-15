---
round: 3
ledger:
  - key: interrupt-settle-no-run-to-resolve
    rounds: [1, 2]
    status: resolved
  - key: webster-default-engine-unbounded
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: attach-mismatch-skips-requires
    rounds: [1, 2]
    status: resolved
  - key: r2-misstates-changed-default
    rounds: [1]
    status: resolved
  - key: instruction-files-no-scenario
    rounds: [1]
    status: resolved
  - key: attach-refusal-way-forward-unreachable
    rounds: [2]
    status: resolved
  - key: attach-way-forward-age-window
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: refusal-row-skip-unjustified
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d4-attach-omits-finished-exemption
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 3 resolved the two long-running design keys: the review found no remaining gap in the attach `Requires` check or in the settle-wait failure behaviour.
The attach way forward now names a step that clears the refusal, but F1 shows it clears only once the run record is older than twice `startup_timeout_s`; that is the new key `attach-way-forward-age-window`.
F2 asks D8 to give the reachability reason for skipping a refusal-spec row, F3 asks D4 to carry the finished-run exemption, and F4 asks that D7's Bound be the single statement of when Webster's audit engine can differ.
