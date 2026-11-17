---
round: 3
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
    status: resolved
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
    status: resolved
  - key: card3-repo-root-source-unnamed
    rounds: [1]
    status: resolved
  - key: renderers-named-by-file
    rounds: [1]
    status: resolved
  - key: stale-test-only-allowlist-prompts
    rounds: [2]
    status: resolved
  - key: burler-fix-table-built-with-nil-values
    rounds: [2]
    status: resolved
  - key: verb-fails-on-unmet-requirement
    rounds: [2]
    status: resolved
  - key: compare-fork-deny-gate-names-absent-spec
    rounds: [2]
    status: resolved
  - key: shuttleparity-package-lacks-testmain
    rounds: [3]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: interim-reachability-not-admitted-in-allowlist
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: seat-specs-keeps-claude-model-params
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: build-edit-glyph-lacks-resign-arrow
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: engine-names-have-no-named-constant
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Every key open through round 2 is resolved:
the round-3 review raised none of them, and its focus verdicts and rejected list clear the copy-warning wording,
the fixture's skill layout, the burler fix table, the verb's exit codes and the `Compare` fork-deny gate.
Round 3 raises five new keys.
`shuttleparity-package-lacks-testmain` is BLOCKING: card 6 creates the test package with no `TestMain`, which `tmuxkit.Main` isolation requires.
`interim-reachability-not-admitted-in-allowlist` is the prefix-of-card-order form of the test-only-code guard:
cards 5 and 6 add production functions no main reaches until card 8.
`seat-specs-keeps-claude-model-params` says `Seat.Specs` keeps Model, Effort and Version, so Codex refuses a Claude pin;
its fix states a contract for `Specs`, so it needs a fresh look.
The last two are LOW consistency: `build` lacks the re-sign arrow, and the engine names have no named constant.
