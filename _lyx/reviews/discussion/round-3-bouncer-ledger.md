---
round: 3
ledger:
  - key: advisor-start-failure-notice-before-chair-exists
    rounds: [1, 2]
    status: resolved
  - key: send-notice-needs-idle-chair
    rounds: [1]
    status: resolved
  - key: done-advisor-strand-removed
    rounds: [1, 2]
    status: resolved
  - key: advisor-to-parent-route-unbounded
    rounds: [1]
    status: resolved
  - key: still-live-undefined
    rounds: [1]
    status: resolved
  - key: provider-word-single-source-claims-false
    rounds: [1]
    status: resolved
  - key: added-after-discussion-duplicates-d6
    rounds: [1]
    status: resolved
  - key: decisions-relitigate-rejected-alternatives
    rounds: [1]
    status: resolved
  - key: send-notices-unserialized
    rounds: [2]
    status: resolved
  - key: chair-to-advisor-edge-unbounded
    rounds: [2]
    status: resolved
  - key: notice-stop-not-off-gate-boundary
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: shuttle-send-unserialized-against-notice
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: resume-removes-finished-advisor-silently
    rounds: [3]
    status: open
    class: design
    severity: LOW
---

Round 1 raised eight findings, round 2 four, round 3 three; the highest severity fell from BLOCKING to MEDIUM.

All ten keys from rounds 1 and 2 are resolved.
Round 3's focus verdicts record the resume of a Done advisor, advisor ends before the chair's handle exists, concurrent notice senders and the chair-to-advisor authority edge as settled.

Round 3 opened three new keys, all about the notice path into the chair's pane.
`notice-stop-not-off-gate-boundary` (F1, MEDIUM): the outputs-exist stop is read before `Run.Send`'s idle wait, so a notice can type at the gated Done boundary beside the chair's own gate send.
`shuttle-send-unserialized-against-notice` (F2, LOW): an external `lyx shuttle send` into the chair is not serialized against the notice sender.
`resume-removes-finished-advisor-silently` (F3, LOW): a resume's probe stops a finished advisor's kept strand without telling the chair.
