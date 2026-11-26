---
round: 1
ledger:
  - key: host-attach-runs-hub-config-binaries
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: area-root-sockets-replaceable
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: attach-client-yama-ancestry
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: area-shares-host-go-caches
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: host-verbs-reach-area-hubs-via-machine-record
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: area-root-outside-lymtemp
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: supervisor-server-env-hygiene
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: restriction-order-and-cgo-tsync
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: cross-area-sender-refusal-fails-open
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: daemons-cannot-read-other-supervisor-records
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: derived-creel-path-unchecked
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: loopback-services-reachable-unclaimed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: run-scope-kill-and-cgroup-report
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: cap-memory-admits-host-side-children
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: codex-home-links-in-area
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: relay-runproc-registration-contradiction
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: relay-acts-on-area-written-state
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: area-deploy-runs-area-code-on-host
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: dead-supervisor-tree-has-no-stop
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: area-stop-trusts-agent-written-run-records
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: start-check-skips-derived-write-grants
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: area-init-lists-contradict-grants
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: apparmor-hash-versus-derived-list
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: open-risk-sender-confirmation-settled
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: open-risk-name-lookups-deferred
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unpinned-plugin-cache-and-credential-helper-facts
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1 is the first round, so every key is new and open.

The findings cluster in a few areas:

- Host-side code that trusts what agents write: attach, host verbs over area hubs, `lym area stop`,
  the relay, `lym area deploy`, the Go caches, and the derived creel paths.
- Peer and process identity across the area boundary: the sockets under the area root, the attach client under Yama,
  creel sender refusal, supervisor records and `cap-memory`.
- Supervisor lifecycle and environment: server environment, restriction order, a dead supervisor's tree and the run scope.
- Grant lists and records that disagree with each other or with the code.
