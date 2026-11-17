---
round: 1
ledger:
  - key: by-slug-beam-start-claims-unowned-prime
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: terminal-summary-hides-attach-and-hold
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: failed-dobby-reads-as-started
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: beam-start-child-boundary-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: dobby-launcher-rows-miss-owner-gate
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: start-window-routing-and-fallbacks-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: hubcatchup-launcher-wiring-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: catchup-test-needs-keep-marker
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: shortname-refusal-speaks-for-creel
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: wsl-launcher-under-refuses-shell-specials
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: launcher-upkeep-races-across-hubs
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: linux-desktop-dir-read-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: pattern-test-isolation-misses-xdg-desktop-dir
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: sandbox-exclusion-reason-stale
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised fourteen findings, all open; there is no earlier ledger.
The gating ones are the two BLOCKING design findings, the by-slug `lym beam start` child claiming an unowned prime and the readable terminal summary hiding the attach commands and the hold, plus three MEDIUM design findings: a failed dobby reported as started, the untested child boundary, and dobby launcher rows that cannot fail on the owner gate.
The rest are LOW test-coverage, refusal-wording, WSL quoting, launcher-upkeep race and documentation findings, and one NIT on a stale sandbox exclusion reason.
