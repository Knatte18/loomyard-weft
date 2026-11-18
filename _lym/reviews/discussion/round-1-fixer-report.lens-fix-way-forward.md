```yaml
lens: fix-way-forward
outcome: reported
```

Task: check the ways forward that the fix added to `_lym/discussion/decision-record.md` D3 and D4: can each be reached from the state it names, and does it lead to progress.

- **MEDIUM.** Changed text in D4: "On a tree dirty before or after the stop it keeps waiting, with the session up for the agent to commit or discard, and does not reroute."
  On a send-back, nobody is left to act, so the row waits with no end. The verb has already told the agent to end its turn, so the agent is idle when the wake finds the tree dirty. Nothing sends it a new prompt, and the row raises no Stuck for the parent.
  The continue-flag case this copies works differently. There the operator has asked the agent to clean the tree and continue, so a live agent is at work.
  Evidence: `internal/landingshed/prreview.go:665-680` resumes a stopped session only when `judge` does not end the review. Resuming a session gives it no instruction. D4 says the verb "answers the agent to end its turn".
  The text needs a way forward: the wake prompts the agent to commit or discard, or it halts for the parent and names commit or discard and `lym loom resume`.
- **LOW.** Changed text in D4: "added to those the verb answered fix-here earlier in the PR-Review episode", and the Bound bullet "one episode fixes at most `pr_review_small_max` findings in place".
  The count is said to cover one episode, but only the send-back reroute clears it, and it lives in the pull-request record, which outlasts the episode. A PR-Review that re-enters by any other route inherits the old count, and its first small set can trigger a send-back.
  Evidence: D4 says "On a clean tree it clears the mark, … and its fix-here count". No other clear is stated, and `recordPullRequest` (`prreview.go:445`) leaves existing fields alone on a found record.

Calls made:
- I read `internal/landingshed/prreview.go:280-300` and `655-700` because my context lacked the wake's stop, re-read and resume path. It decides whether "the session up" holds and whether anything prompts the agent.
