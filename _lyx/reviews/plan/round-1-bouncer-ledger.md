---
round: 1
ledger:
  - key: step-entry-observation-before-bootstrap-spends-marker
    rounds: [1]
    status: resolved
  - key: no-test-of-completed-step-then-step-handoff
    rounds: [1]
    status: resolved
  - key: bound-omits-crash-resume-after-refused-or-errored-step
    rounds: [1]
    status: resolved
  - key: card10-missing-cancelled-and-unreadable-status-test-arms
    rounds: [1]
    status: resolved
  - key: selfreport-github-test-doc-goes-false
    rounds: [1]
    status: resolved
  - key: halt-note-tier1-line-unnamed-residual
    rounds: [1]
    status: resolved
  - key: halt-note-line-and-stencil-wording-mismatch
    rounds: [1]
    status: resolved
---

Round 1 is the first round of this rework generation, which starts at card 10 after PR-Gate rejected PR 355.
The review raised seven findings and approved:

- two MEDIUM, both on card 11: the step entry observation sat before the bootstrap, so a refused step spent the clean-handoff marker, and no test drove the real completed-step-then-step sequence;
- four LOW: residuals missing from cards 11 and 12's bounds, test arms missing from card 10, and a test doc in `selfreport_github_test.go` that would go false;
- one NIT: the halt-note line and the stencil sentence were worded differently.

The fixer then rewrote cards 10, 11 and 12, and all seven are resolved in the live plan.
No new finding arose from the judge's own pass.
