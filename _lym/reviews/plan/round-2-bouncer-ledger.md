---
round: 2
ledger:
  - key: assist-seat-escapes-catalog-and-scan
    rounds: [1]
    status: resolved
  - key: seats-func-test-only-caller
    rounds: [1]
    status: resolved
  - key: manifest-omitted-no-production-caller
    rounds: [1]
    status: resolved
  - key: unmet-skip-narrows-d1-unrecorded
    rounds: [1]
    status: resolved
  - key: parity-fixture-lacks-seat-skills
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: card6-refusals-reached-only-by-card8
    rounds: [1]
    status: resolved
  - key: standing-passages-test-repeats-existing
    rounds: [1]
    status: resolved
  - key: copy-bound-misstates-start-retry
    rounds: [1]
    status: resolved
  - key: loom-driver-omitted-from-unmet-producers
    rounds: [1]
    status: resolved
  - key: one-way-forward-for-two-copy-failures
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card3-repo-root-source-unnamed
    rounds: [1]
    status: resolved
  - key: renderers-named-by-file
    rounds: [1]
    status: resolved
  - key: stale-test-only-allowlist-prompts
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: burler-fix-table-built-with-nil-values
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: verb-fails-on-unmet-requirement
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: compare-fork-deny-gate-names-absent-spec
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's three BLOCKING design keys and its decision key are resolved:
the round-2 review's rejected list confirms the assist seat reaches the catalog and scan, the plan uses `seatsOf`,
`Manifest.Omitted` has a production caller through `WriteManifest`, and the plan carries no unmet-requirement skip.
Round 1's card-6 refusal, standing-passage test, copy-bound, loom-driver and renderer-naming keys are resolved by the same list;
`card3-repo-root-source-unnamed` drew no round-2 finding and is resolved.
Two round-1 keys continue as narrower findings:
the seat-skill fixture (`parity-fixture-lacks-seat-skills`, now about the skill layout and where seat skills come from)
and the copy-warning ways forward (`one-way-forward-for-two-copy-failures`, now about several causes per error).
Round 2 adds two BLOCKING keys: the stale `internal/seatengine.Prompts` allowlist entry, and the nil-valued burler fix table.
It also adds the verb's non-zero exit for an unmet requirement (MEDIUM, scope)
and the `Compare` fork-deny gate that names a spec it is not given (LOW, consistency).
