# A beam pull request updates its description after each member landing

`beam-pr-describes-itself` · Note · enhancement, dobby, landing

## Seen

While a beam's pull request is recorded, each member landing asks the orch, through its landed line, to add the landing to the change description and run `lym beam describe <slug>`.
The orch gets that line only when the member's `land_notice` is on (`member-line-honours-land-notice`), so with notices off the pull request's description falls behind.

## Idea

Dobby, or the landing itself, updates the beam's change description and runs the `lym beam describe` step when a member lands under a recorded pull request.
No orch is asked, and the landed line drops its follow-up.
