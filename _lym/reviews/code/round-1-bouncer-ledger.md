---
round: 1
ledger:
  - key: teardown-archive-exclusion-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: records-tag-vs-records-branch-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: here-archives-cli-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: resumed-record-flag-merge-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: here-refusal-rows-missing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: read-owner-archives-branch-rows-missing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: shortcuts-test-keep-marker
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: remote-url-error-read-as-no-remote
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: adoptcli-doc-flag-list-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised nine findings, all open.
Two are gating MEDIUM test-coverage gaps in `ReadOwnerArchives` (teardown `archive/` tags, records branch comparison).
The rest are LOW: missing test rows for the `--here` and `--archives` wiring, refusals and resume merge, a keep marker,
an over-broad `RemoteURL` error swallow, and a stale package doc sentence.
