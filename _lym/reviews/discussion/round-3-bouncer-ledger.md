---
round: 3
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
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
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
    status: resolved
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
    rounds: [2, 3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: nested-fake-hub-escapes-area-refusal
    rounds: [2]
    status: resolved
  - key: area-hub-identity-from-area-written-files
    rounds: [2]
    status: resolved
  - key: window-title-reaches-konsole-arguments
    rounds: [2]
    status: resolved
  - key: host-hub-selection-bypasses-area-check
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: area-read-grants-cannot-serve-cross-hub-mail
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: boot-builds-sibling-landlock-domain
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: attach-client-carries-host-env-and-terminal
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: local-marketplace-inside-area-hub
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: host-processes-follow-area-made-links
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: slack-records-on-wrong-side-of-boundary
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: apparmor-barrier-departs-from-closed-area-pattern
    rounds: [3]
    status: open
    class: consistency
    severity: BLOCKING
    fix: design-wrong
  - key: pattern-backgrounds-contradicted-without-declaration
    rounds: [3]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
---

Round 1 raised a broad set of keys, and round 2 closed all but the relay bound and the AppArmor profile text.

Round 2 added the inside test, the nested fake hub, the spoofed hub identity and the window title.

Round 3 closes the AppArmor profile text, the nested fake hub, the hub identity and the window title.
The inside test stays open on a new route: a process can leave its scope through `clone3`.
The relay bound stays open on one missing effect: channel retirement.

The new round 3 keys fall in three groups.
Exits the area leaves open: host verbs that take a hub from a path argument, the attach client, a local marketplace
and host writes through area-made links.
Mechanisms that do not work as described: the cross-hub read grants, the second Landlock domain and Slack's record placement.
Departures from PATTERN texts: the AppArmor barrier and two background rules.

Not findings, held back by the review's incremental rule:
the `/dev/pts` grant and the unsourced `lym area deploy` origin and branch (both rated BLOCKING in weight by the reviewer),
no removal of a hub's area-registry entry when its area is turned off,
the filter flags lacking `TSYNC_ESRCH`,
and the area-file key list lacking the deploy command.
