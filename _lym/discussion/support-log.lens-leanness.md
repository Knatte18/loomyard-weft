```yaml
lens: leanness
outcome: reported
```

Lens: leanness. I looked for content to cut or move out of the decision record, and found nothing BLOCKING.

**MEDIUM — `## Open risks`, both bullets.**
These are trade-offs the design accepts, not questions only the parent can settle.
- The first bullet (no operator in PR-Review or Discussion under autonomy) follows from D3 and D4, which already state it.
- The second bullet (the orch's design picks reach no operator before landing) follows from D5 and D7.
- Neither names an entry, run or commit, which is what Step 4 reserves this section for. The plan writer needs neither one to act.
- Fix: delete both and leave a short line saying no open risks remain, or move the reasoning to the support log's Rejected alternatives or Interview.

**LOW — `## Auto-mode assumptions`, bullet "`code_review_ask_parent` stays its own key under autonomy (D3)".**
D3 already decides this and gives the reason. Cut it.

**LOW — `## Auto-mode assumptions`, bullet "A beam's own Finalize halts follow D5 under the beam's own value".**
This is a decision, not an assumption. D1 already says a worktree's value governs "for a beam, the beam's own landing steps". Cut it, or add a clause to D1 or D5 if the Finalize halt case needs naming.

**LOW — `## Scope` Out bullets repeat decisions.**
- "Turning member pull requests off…" restates D4's first sentence and the end of D4's Rationale.
- "Merging the beam's pull request: Finalize still merges it after `lym beam ready`" restates D7's last line and its Rationale.
- Fix: keep the Out bullets as one-line exclusions without the mechanism, or cut the repeated sentence from D4's Rationale ("the operator who wants no member pull requests sets `pull_request` off on the beam").

**LOW — `## Constraints` repeats decision rationales.**
- The PATTERN-landing, PATTERN-github-auth and PATTERN-daemon-wakeups lines restate what D4, D6 and D7 already argue from the same entries.
- D6's Rationale "already wakes at most once a minute" duplicates the PATTERN-daemon-wakeups constraint.
- Fix: say each fact once. Keep the constraints, since they bind the plan writer, and trim the matching rationale clauses.

**LOW — `## Notes for the plan writer`.**
- It repeats `relevant.md`: `parentPullRequestMode` and `internal/loomcli/parentseatwiring.go` both appear there, and `parentPullRequestMode` also appears in D2's Rationale.
- The spec says the list of code read belongs in `relevant.md`.
- Fix: cut the second bullet, which only says D3 leaves the file alone, a non-change. Trim the first to the seam hint alone (`ParentReviewWanted`, `JudgeAskParent`).

**LOW — `## Goal`, list item 1.**
"which the orch cannot settle" is redundant beside "one of the closed classes D5 names". The two phrasings could also be read as two separate tests. Keep one.

Other things I noticed:
- **Procedure, not a record defect:** the writer spawned only this leanness fork. The lens procedure requires all six lenses in one message. The interactions, existing-paths, bound-breaker, walkthrough and board-scope forks were never spawned. The writer should record this at the support log's lens review.
- **Constraints, last bullet:** "shellguard denies none of them" is a claim about existing code, but it names no file. The pin is `internal/shellguard/rows.go`, the source the explore subagent read. Without it, a later round must check the claim again.

Tool calls: none; my context held everything this lens needed.
