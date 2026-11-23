---
round: 2
focus:
  - "internal/loomcli/parentreview.go, `parentReviewState.preRun` and the `parent-review ask` verb: settle whether a test fails when the verb stops passing the session's own strand (`LYM_STRAND_NAME`) to `prReviewAsker`, the claim that issue #200 turns on."
  - "_lym/landing/summary.md `## Merge danger`: settle whether the call and blast radius name the new `undelivered` state the diff writes into `.lym/parent-review/asks.json`, and the seats that write it (PR-Review ask, the drift judge's ask in `internal/driftengine/judgeparent.go`, the review rounds' parent seat in `internal/shedadapters/parentseat.go`)."
  - "contracts/stencils/landing/landing-block-pr-review-autonomous.md and the `ask` verb's Long text in internal/loomcli/parentreview.go: settle whether the wording of what an undelivered end means matches `ParentAskUndelivered`'s doc, which covers a refused or failed delivery."
---

The three entries name the sites the diff changes around the undelivered ask end; the reviewer still reviews the whole range.
