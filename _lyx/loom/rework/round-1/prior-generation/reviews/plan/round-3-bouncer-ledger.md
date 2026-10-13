---
round: 3
ledger:
  - key: failed-client-listing-guard-test-unreachable
    rounds: [1]
    status: resolved
  - key: lyx-hidden-doc-claim-goes-stale
    rounds: [1]
    status: resolved
  - key: placed-new-window-double-target-flag
    rounds: [1]
    status: resolved
  - key: overview-anchor-unbounded-grouped-display
    rounds: [1]
    status: resolved
  - key: new-agent-window-selected-pinned-twice
    rounds: [1]
    status: resolved
  - key: hidden-placement-pinned-by-pure-and-engine-row
    rounds: [1, 2]
    status: resolved
  - key: new-window-locked-two-homes
    rounds: [1]
    status: resolved
  - key: replace-strand-never-refused-too-broad
    rounds: [1]
    status: resolved
  - key: open-window-moves-literal-stale-allowlist
    rounds: [2]
    status: resolved
  - key: doc-go-sentences-contradict-new-code
    rounds: [2, 3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: open-window-nil-state-and-argv-unstated
    rounds: [2]
    status: resolved
  - key: overview-window-size-only-refusal-untested
    rounds: [2]
    status: resolved
  - key: shell-window-of-outside-selvage-seam
    rounds: [3]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: subtest-name-contradicts-new-assertion
    rounds: [3]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised eight findings, four of them gating design MEDIUMs; all are resolved by round 3.
Round 2 added the stale `windowCreationAllowlist` entry, the `doc.go` sentences, and two LOW gaps; only the `doc.go` key is still raised.
Round 3 narrows the `doc.go` key to three further sentences (LOW) and adds two new keys.
The BLOCKING one is card 5 placing `shellWindowOf` in `windoworder.go`, which `TestSelvageIdentifiersConfinedToSelvagePane` rejects; the fix moves it to `selvagepane.go`.
The NIT is a subtest name that contradicts its new assertion.
No design-class finding is open.
