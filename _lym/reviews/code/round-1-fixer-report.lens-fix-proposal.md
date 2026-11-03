```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: challenge the proposals of F1–F4 as carried out.

**F1 — MEDIUM.** The change is `resolveWithParentRule` in `pullrequestmode.go`, which now halts on any non-transient, non-token rule-read failure when `recordFound`.
- **Case:** a run holds a record, but the pull request was opened for another reason. Examples are the entry's `pull_request: on`, or a beam.yaml `on` that was later changed to `off`.
- With beam.yaml `off`, `ResolvePullRequest` answers no. The rule read then runs, and a 403 or 404 (for example a token without the rules scope) now halts every Finalize and PR-Review re-entry.
- Before the fix, the same case landed locally or closed the pull request. Under `off` with no rule, that outcome was the intended one.
- `resolveWithParentRule` cannot tell "a pull request opened because of the rule" from "opened for another reason". The record carries no such fact.
- `client.go:219-224` returns every `GetRulesForBranch` error unmapped, so a 404 is a real failure and not a "no rules" answer.
- The halt text says "fix what GitHub's message names", and a persistent 403 or 404 gives the operator no way out except `lym batten cancel`.
- The fix is fail-safe for the goal's bug. The cost is a halt in a case where the rule never applied. This is a tradeoff the review did not weigh, and the report should record it.

**F2 — LOW.** The change is the new `rec.ReviewEnded` case in `judge`'s `StateOpen`, which returns `done()` on a clean tree.
- It does not check `gated`. The `Continue` path just below it requires `clean && gated`.
- If the head moves during the wait, `wake` gates it first. A gate failure leaves the head ungated and mailed, and the tree stays clean.
- `judge` then returns Done for an ungated head, and Finalize merges on GitHub from the pushed head.
- Reaching this needs a commit during an ended review with no session running, so it is unlikely.
- The old closed-pull-request case has the same gap, so this is consistent with it, but it is not safe.

**F3 — LOW.** The cancelled-context subtest in `prreview_test.go` scripts `context.Canceled` as the read's error.
- `shedtransient.Class` probably marks `context.Canceled` transient. If so, removing the `cancelErr` check inside `transientFailure` would still yield an error wrapping `context.Canceled`, and the subtest would still pass.
- So the row pins "an error is returned", and not specifically the cancel path. I did not verify the classifier.

**F4:** no findings. An early return that is dropped would show up as a `RequiresPullRequest` call in `wantCalls`.

Further tool calls (1 of 6):
- `grep` of `RequiresPullRequest` in `internal/pullrequest/client.go`, to see how the client maps a 404 or 403. My context only had the fake.
