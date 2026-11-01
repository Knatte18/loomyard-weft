---
round: 2
ledger:
  - key: crucible-smoke-commands-tags-only
    rounds: [1]
    status: resolved
  - key: tmuxallowed-unpinned
    rounds: [1]
    status: resolved
  - key: require-parse-failure-untested
    rounds: [1]
    status: resolved
  - key: testtiming-setenv-unpinned
    rounds: [1]
    status: resolved
  - key: require-reading-paths-fixtures
    rounds: [1]
    status: resolved
  - key: battencli-tier0-push-refusal-lost
    rounds: [1]
    status: resolved
  - key: stale-tag-wording
    rounds: [1]
    status: resolved
  - key: main-pass-counts-missing
    rounds: [1]
    status: resolved
  - key: template-live-tests-by-build-tag
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised eight findings, one of them gating (`tmuxallowed-unpinned`).
Round 2 found none of them again and raised one new LOW consistency finding: the review template still tells the instantiator to find live tests by a `go:build` tier tag.
Its fix is a local wording change the finding text already specifies.
