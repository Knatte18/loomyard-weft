---
round: 2
focus:
  - "internal/loomcli/parentreview.go `parentReviewAskVerb`: with `landingshed.ReadPRReviewStanding` returning satisfied at the head and a later ask at that head still open, does the verb refuse without recording or posting, with a message naming `lym loom pr-continue`, and does it still refuse on `PRReviewAwaiting`?"
  - "contracts/stencils/landing/landing-block-pr-review-autonomous.md: with a satisfied answer standing at the head and a later ask ended timeout, no-parent or not-sent, does every rule send the session to `lym loom pr-continue` rather than to ask again, and does each `It` read against its intended antecedent?"
  - "internal/landingshed/prreviewask.go and internal/loomcli/doc.go: do the `PRReviewAwaiting` doc and the ask verb's doc line state the satisfied precedence the code implements?"
---

Focus restates the sites the round-1 findings touched, as questions for a fresh read of the range.
