---
verdict: APPROVED
rationale: "The round-2 fixer rewrote card 8 after the review was saved, and all three findings are fixed against internal/landingshed/publish.go. checkApproval's FindPullRequest (publish.go:248) is the fifth arm, with a decided:true error that Call returns at publish.go:117 before any merge-in or push, plus a 503/422 test. Each arm calls cancelErr before the classified branch, with a cancelled-context test. Worktree-Teardown's RemoteBranchError is named out of scope with a reason. No open findings remain."
---

# Plan-Review round 2 — judge verdict

The round-2 review was saved at 17:20:11.
The fixer then rewrote `_lyx/plan/08-producer-transport-errors.md` at 17:20:33, and it touched no other card.
So I judged the current card 8 against the review's three findings and against the code.

## F1 (BLOCKING): approval-path pull-request query

Resolved.
Card 8's Intent now names `checkApproval`'s `FindPullRequest` (`publish.go:248`) as a fifth arm, the first GitHub call after every `lyx loom approve`.
The card returns the classified error with `decided: true`.
`Call`'s step 3a (`publish.go:116-118`) returns on `decided || err != nil`, so the error comes back before any merge-in or push, as the card says.
The new test puts an approval record on disk, sets a `TaskHead` and has the list call answer 503.
It expects an error classified `github-api`, the warn line, and no merge-in or push.
A 422 on the same call keeps `Stuck`.
The ImpactSummary now counts five arms, and the re-entry paragraph says a re-step runs the approval check first.

## F2 (LOW): cancellation before the classified branch

Resolved.
The card no longer claims the arms have their own cancellation check.
It states correctly that none of the five arms checks its context before `stuckOrCancelled`, which is where `cancelErr` runs (`publish.go:291-292`, `finalize.go:311-312`).
So the new branch calls `cancelErr(ctx, <row>)` first in each arm, which keeps `ctx.go`'s rule.
The added test sends a transport `*GitError` under an already-cancelled context.
It expects an error wrapping `context.Canceled` whose class is empty.

## F3 (LOW): Worktree-Teardown remote deletion

Resolved.
The out-of-scope list now names batten's Worktree-Teardown.
The reason is sound: `fabricengine.Remove` reports a failed deletion as text in `RemoveResult.RemoteBranchError`, so the error has no chain to classify, and adding one would widen `fabricengine` beyond what the decision record licenses.

## Focus departures

The round-2 report has no `## Focus departures` section, so there is nothing to ratify.
The review's process note says the reviewer read round-1 fixer and judge files before the instructions allowed it.
It independently confirmed F1 and F2 against the code, and F3 is its own new finding, so the early read changed no finding.

## Other checks

Card 8 still targets only `landingshed`, whose tests the `## verify:` section runs both untagged and with `-tags integration`.
I found no new granularity, typing or decision-record fidelity issue in the rewritten card.
