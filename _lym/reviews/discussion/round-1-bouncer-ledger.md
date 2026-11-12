---
round: 1
ledger:
  - key: bridge-cursor-start-loses-first-message
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: lost-settle-past-hour-returns-delivered-mail
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: bridge-posts-wiped-from-outbox-record
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: bridge-channel-commands-run-on-both-machines
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: resync-adoption-trusts-any-envelope-poster
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: unmapped-channel-events-trigger-full-resyncs
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: first-contact-receiver-uninformed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: acceptance-criteria-miss-stated-behaviours
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d4-holds-versus-d7-refusal-cap
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: once-per-pair-listing-false-on-hold-path
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: program-claim-bound-linux-only
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: from-naming-this-machine-held-forever
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: mail-to-retired-orch-through-recreated-bridge
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: app-id-lowercased-by-command-matching
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: machine-inbound-mail-in-creel-other-hub-branches
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: former-name-failing-checkname
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, so no earlier ledger exists and every key is new.

The gating cluster is the bridge channel's read and post lifecycle in D3, D4, D7 and D8:

- where the read cursor starts on adoption and resync (the BLOCKING key);
- what creel does with a settle that arrives after the hour;
- where bridge posts are kept so orch-mail reads do not wipe them;
- who handles operator commands in a bridge channel;
- whom resync adoption trusts;
- what an event in an unmapped channel can trigger;
- what the receiving operator is told on first contact.

The remaining keys are local corrections or small added rules.
The review's single rejected item (no wake for a cleared hold) was ruled out on the take loop's own backoff and is not carried.
