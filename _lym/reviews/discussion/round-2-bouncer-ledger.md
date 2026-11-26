---
round: 2
ledger:
  - key: host-attach-runs-hub-config-binaries
    rounds: [1]
    status: resolved
  - key: area-root-sockets-replaceable
    rounds: [1]
    status: resolved
  - key: attach-client-yama-ancestry
    rounds: [1]
    status: resolved
  - key: area-shares-host-go-caches
    rounds: [1]
    status: resolved
  - key: host-verbs-reach-area-hubs-via-machine-record
    rounds: [1]
    status: resolved
  - key: area-root-outside-lymtemp
    rounds: [1]
    status: resolved
  - key: supervisor-server-env-hygiene
    rounds: [1]
    status: resolved
  - key: restriction-order-and-cgo-tsync
    rounds: [1]
    status: resolved
  - key: cross-area-sender-refusal-fails-open
    rounds: [1]
    status: resolved
  - key: daemons-cannot-read-other-supervisor-records
    rounds: [1]
    status: resolved
  - key: derived-creel-path-unchecked
    rounds: [1]
    status: resolved
  - key: loopback-services-reachable-unclaimed
    rounds: [1]
    status: resolved
  - key: run-scope-kill-and-cgroup-report
    rounds: [1]
    status: resolved
  - key: cap-memory-admits-host-side-children
    rounds: [1]
    status: resolved
  - key: codex-home-links-in-area
    rounds: [1]
    status: resolved
  - key: relay-runproc-registration-contradiction
    rounds: [1]
    status: resolved
  - key: relay-acts-on-area-written-state
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: area-deploy-runs-area-code-on-host
    rounds: [1]
    status: resolved
  - key: dead-supervisor-tree-has-no-stop
    rounds: [1]
    status: resolved
  - key: area-stop-trusts-agent-written-run-records
    rounds: [1]
    status: resolved
  - key: start-check-skips-derived-write-grants
    rounds: [1]
    status: resolved
  - key: area-init-lists-contradict-grants
    rounds: [1]
    status: resolved
  - key: apparmor-hash-versus-derived-list
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: open-risk-sender-confirmation-settled
    rounds: [1]
    status: resolved
  - key: open-risk-name-lookups-deferred
    rounds: [1]
    status: resolved
  - key: unpinned-plugin-cache-and-credential-helper-facts
    rounds: [1]
    status: resolved
  - key: inside-test-fails-open-without-ancestry
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: nested-fake-hub-escapes-area-refusal
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: area-hub-identity-from-area-written-files
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: window-title-reaches-konsole-arguments
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 2 closed every round 1 key but two:
the relay bound still understates what the host relay does with area-written input,
and the AppArmor profile text still changes when a new area hub appears.

The four new BLOCKING keys are of two kinds.
Three let area-written input decide what a host process does or whom it trusts:
the nested fake hub, the spoofed repository identity and the window title.
The fourth is the inside test, which fails open when a filtered process loses readable ancestry.
It is a new key beside the resolved sender-refusal key, whose original cases are handled.
