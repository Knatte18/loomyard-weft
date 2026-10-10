---
round: 3
ledger:
  - key: socket-check-runs-after-sweep
    rounds: [1]
    status: resolved
  - key: reed-servers-unregistered-with-tmuxkit
    rounds: [1]
    status: resolved
  - key: reed-exit-empty-probe-unlicensed
    rounds: [1]
    status: resolved
  - key: pattern-overview-cap-uncarried
    rounds: [1]
    status: resolved
  - key: exit-empty-early-return-false-bound
    rounds: [1]
    status: resolved
  - key: warplayout-test-not-read-only
    rounds: [1]
    status: resolved
  - key: prebuilt-variable-not-stripped-on-skip
    rounds: [1]
    status: resolved
  - key: package-docs-state-false-bounds
    rounds: [1]
    status: resolved
  - key: rev-parse-survivors-omits-mergeheadpresent
    rounds: [1]
    status: resolved
  - key: prestarted-server-bypasses-reed-start-contract
    rounds: [2]
    status: resolved
  - key: socket-check-misses-reed-removed-socket
    rounds: [2]
    status: resolved
  - key: registration-net-runs-no-test
    rounds: [2]
    status: resolved
  - key: reboot-after-teardown-configless-server
    rounds: [2, 3]
    status: open
    class: decision
    severity: MEDIUM
  - key: forged-prebuilt-bound-false-for-plan-verify
    rounds: [2, 3]
    status: open
    class: scope
    severity: MEDIUM
  - key: integration-or-tmux-passes-tier-purity-scan
    rounds: [2]
    status: resolved
  - key: nested-module-claim-contradicts-buildroot
    rounds: [2]
    status: resolved
  - key: card7-restates-struck-only-copy-claim
    rounds: [2]
    status: resolved
  - key: weight-report-llm-tier-unmeasured
    rounds: [2]
    status: resolved
  - key: socket-dir-rule-fails-client-probe-packages
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
  - key: tierpurity-platform-tags-by-host-goos
    rounds: [3]
    status: open
    class: design
    severity: LOW
---

Round 1 raised nine findings, all resolved by round 2.
Round 2 raised nine findings; round 3 no longer raises seven of them.
Round 3 reopens two: the config-less reboot bound (decision) and the prebuilt variable's reach into webster's card-verify rerun (scope, same key as the round 2 forged-prebuilt bound).
Round 3's BLOCKING finding is new: card 10's per-user directory rule fails tier-2 packages such as `fabriccli` and `battencli` whose tests reach a tmux client probe through `pairteardown`.
The other new finding is LOW: the tmux-tier test evaluates platform tags against the host's GOOS.
