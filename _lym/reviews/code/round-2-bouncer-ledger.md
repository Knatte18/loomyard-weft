---
round: 2
ledger:
  - key: by-slug-beam-start-claims-unowned-prime
    rounds: [1]
    status: resolved
  - key: terminal-summary-hides-attach-and-hold
    rounds: [1]
    status: resolved
  - key: failed-dobby-reads-as-started
    rounds: [1]
    status: resolved
  - key: beam-start-child-boundary-untested
    rounds: [1]
    status: resolved
  - key: dobby-launcher-rows-miss-owner-gate
    rounds: [1]
    status: resolved
  - key: start-window-routing-and-fallbacks-unpinned
    rounds: [1]
    status: resolved
  - key: hubcatchup-launcher-wiring-unasserted
    rounds: [1]
    status: resolved
  - key: catchup-test-needs-keep-marker
    rounds: [1]
    status: resolved
  - key: shortname-refusal-speaks-for-creel
    rounds: [1]
    status: resolved
  - key: wsl-launcher-under-refuses-shell-specials
    rounds: [1]
    status: resolved
  - key: launcher-upkeep-races-across-hubs
    rounds: [1]
    status: resolved
  - key: linux-desktop-dir-read-untested
    rounds: [1]
    status: resolved
  - key: pattern-test-isolation-misses-xdg-desktop-dir
    rounds: [1]
    status: resolved
  - key: sandbox-exclusion-reason-stale
    rounds: [1]
    status: resolved
  - key: no-prime-claim-flag-literal-repeated
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
---

Round 1's fourteen findings are all resolved in the round 2 review, F1 to F14, two of them by a fix that differs from the one proposed and holds.
Round 2 raised one new BLOCKING `design` finding, F15: the hidden `no-prime-claim` flag name is spelled as a literal in beamcli's `startCmd` and again in machinecli, with nothing tying the spellings together.
The fix is one exported constant in beamcli, used by machinecli and the beamcli flag test.
The review rejected an assistant finding about the launcher lock's placement, F16, on the `hubregistry` `hubs.lock` precedent.
