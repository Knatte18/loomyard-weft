# landingshed: a failed later ask no longer hides a satisfied answer

Under autonomy, `lym loom pr-continue` judged a review by the latest PR-Review ask at the head.
A later ask that never reached the parent, ended `undelivered`, `no-parent` or `timeout`, or still awaited an answer,
hid a satisfied answer the parent had already given at that head.
`pr-continue` then refused with "no satisfied answer from your parent stands", and the review deadlocked.

Closes #213

## What changes

- `ReadPRReviewStanding` in `internal/landingshed/prreviewask.go` now looks for the latest answered, uncleared PR-Review ask at the head.
  When its report holds no findings, the standing is satisfied, whatever asks came after it at that head.
  A later answer with findings replaces it, and a cleared answer does not stand.
  With no answer standing, the latest ask decides as before: awaiting before its deadline, ask-again otherwise.
- `lym loom parent-review ask` refuses an ask at a head where a satisfied answer stands,
  and its refusal names `lym loom pr-continue` as the way forward.
  This stops the agent from posting more asks that cannot change the outcome.
- The `landing-block-pr-review-autonomous` stencil tells the PR-Review agent that a standing satisfied answer outlasts later failed or unanswered asks,
  and to run `pr-continue` instead of asking again.
- The `landingshed` and `loomcli` package docs state the new rule.

## Design decisions

- The parent's answer at a head holds until the parent answers again at that head.
  This is the one rule behind both the standing read and the ask refusal.
- The change adds no verb for the parent to record a satisfied answer without a delivered ask, and no `--head` flag.
  With the standing fixed, a satisfied answer the parent already gave is enough for `pr-continue`.
- The tmux-tier test that runs `parent-review ask` from a live non-orch pane is out of scope here and filed as its own board note.
  The strand claim creel accepts from a live PR-Review pane therefore stays unproven.

## To check by hand

- `parent-review ask` posts a second ask past a satisfied answer in a case the new refusal does not cover (`ask-verb-posts-second-ask-past-satisfied`).
  Check `parentReviewAskVerb` in `internal/loomcli/parentreview.go` for a path that reaches the post without the standing check.
- The stencil's await rule can still send the agent to ask again past a satisfied answer (`stencil-await-rule-asks-again-past-satisfied`).
  Read the `await` paragraph of `contracts/stencils/landing/landing-block-pr-review-autonomous.md` against the new `pr-continue` instruction.

## Merge danger

**Door:** two-way
**Blast radius:** landingshed
