---
round: 2
focus:
  - "Slot waits in status: the decision record's D6 says a Go-side site's wait is recorded as a waiting state visible in status, and D4 lists the card `**Verify:**` rerun and the plan gate's `go list` load as Go-side sites that acquire a slot. Check whether every plan card that acquires a slot through `Pool.Acquire` outside `verifytree.Verify` (the card-verify rerun card and the plan-gate card) records a wait that the status hooks card reads. Settle whether a gate stalled behind a Publish shows as waiting in `lyx loom status`."
  - "Inheritance routing edge: card 10 lets `lyx gate test` run without acquiring when `gateslot.InheritEnv` names a slot that `Pool.Inherited` reports held. Check that the card introducing it states in its `**Intent:**` what a forged variable lets through and what bounds it, per the attack-surface question and D4's and D7's Bounds."
  - "Docs in the same commit: CLAUDE.md requires the module `doc.go` to follow an observable behavior change. Check each `doc.go` of a package whose gate behavior a card changes (`internal/websterengine`, `internal/loomshed`, `internal/impactset`) for a paragraph that still states the replaced behavior, and whether some card lists that file under `Edit:`."
---

Round 1 left three gating design findings; the entries above name the sites to re-check, not the earlier findings.
