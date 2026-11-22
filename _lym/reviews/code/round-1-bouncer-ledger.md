---
round: 1
ledger:
  - key: required-to-on-rewrite-drops-meaning
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: per-task-keys-declared-at-several-sites
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: reinterpret-reads-outside-the-one-lookup
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: task-setting-chain-wrong-lymcwd-api
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: carry-misses-runs-in-flight
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: worktree-files-left-out-of-reinterpretation
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: bad-beam-run-settings-pass-upsert
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: unread-land-notice-dropped-silently
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: carried-legacy-off-overrides-parent
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: replay-does-not-fold-legacy-keys
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: reinterpretation-registration-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: member-line-overrides-explicit-off
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: beam-project-notify-on-land-dropped
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: advisory-read-aborts-reconcile
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: old-clear-owed-memory-second-notice
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: board-skill-prune-rule-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: overview-resolver-input-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: strictness-section-disagrees-with-layers-rule
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: chain-builder-contracts-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: merge-demand-clause-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: contained-tests-lack-keep-markers
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: halt-way-forward-wrong-in-two-cases
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: lookup-skips-envsource-expansion
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: blast-radius-too-narrow
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: seam-names-still-say-parent
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first review of the range: 25 findings, none carried from an earlier ledger.
Four BLOCKING design findings lead: the required-to-on rewrite that claims to keep a meaning it drops,
per-task keys and values declared at several sites, the reinterpretation's read outside the one lookup,
and `TaskSettingChain` using `lymcwd.ResolveWorktree` instead of `lymcwd.NamedWorktree`.
Eight more design findings at MEDIUM gate, mostly about the one-time carry of legacy board fields
(runs in flight, legacy `off`, replay, projects) and gaps in the reinterpretation's reach.
F12 is a `decision` finding deferred to the parent and does not gate.
The rest are LOW and NIT: docs, tests, keep markers and names.
