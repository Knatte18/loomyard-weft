```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether the changes agree with their sibling decisions, criteria and cross-references.

**LOW — D2's base source now names only the reach step, and the twin callers lost theirs.**
- **Changed text:** `_lym/discussion/decision-record.md`, D2, the last bullet: "The reach step hands in the files at the merge base with the parent branch, …, as the base, and the worktree's files as the head."
- **What is stale:** The fix removed D2's general bullet "The diff runs from the merge base with the parent branch, as the redundancy step takes it, to the worktree's files."
  It put back a sentence that names only the reach step.
  D7's failure triage also uses the change set ("the run's changed functions (D2)"), and it runs on a failing call.
  D3's reach step runs only "after its tiers pass", so it never runs on that call.
  D7 therefore no longer has a stated base.
  D7's new bullet ("When the call has no parent branch, or its merge base cannot be read, D2 has no change set") assumes the merge base without saying so.
  D9 is also left with no stated caller ("the same base and worktree files as D2").
- **Evidence:** D3: "runs a step after its tiers pass". D7: "When a `lym gate test` call fails … names the run's changed functions (D2)". D2 now: "The caller hands in both sets".
- **Fix:** Name every caller that uses the merge base against the worktree's files, the reach step and the failure triage of D7, or say "the gate's steps and triage".

No other findings.
The D8 twins of F12 and F14 were both updated.
The F3 twin in Open risks was updated, and the D3 criteria match the reach turn and the linked-package rule.
D5's red-tier rule carries over to D10 through "matched against HEAD as D5 matches".
D9 and D11 agree on the prime case, the D6 refusal matches its criterion, and the D1 stamp matches its criterion.

Further tool calls: none.
