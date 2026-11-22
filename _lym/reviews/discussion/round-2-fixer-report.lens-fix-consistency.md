```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-2 fix agrees with its sibling decisions, criteria and cross-references, and find any change whose twin elsewhere was left unchanged.

**[MEDIUM] D5's dobby tick now reads the code branch, but D3's refused-push gate still reads only the records branch**
- Changed text: D5, "a pair is ahead when its local records branch or its local code branch is."
- Summary: a refused code-side push now restarts the pusher on every dobby tick, not only on every commit, because the gate does not see a refusal on the code branch.
- Evidence: D3 lists among the start conditions "no refused-push record stands for its records branch (D5)". D5's refusal bullet says "While a refused-push record stands for the worktree's branch, neither a commit nor the dobby starts its pusher again". D5's own criterion says "no commit or dobby tick starts that worktree's pusher until the record is cleared".
  - Before this fix, a refused code push left the code branch ahead, but the tick did not count the code branch, so only commits restarted the pusher. The review's Rejected list judged that MEDIUM.
  - With the code branch counted, every tick re-pushes, is refused again and writes a new record and notice, against D5's statement and criterion above.
  - The twin edit is D3's start condition: it should name a refused-push record for the records branch or, for a pair, its code branch.

**[LOW] The support log's Turn 5 still says D7 names the pause record `lym loom pause` writes**
- Changed text: D7's pause sentence, which now names `tendercli.PausePair`.
- Summary: support-log.md Turn 5 says "D7 drops the gate and D7 names the pause record `lym loom pause` writes". That is a dated history entry, and the new Round 2 entry supersedes it. A reader could still take the stale wording for the record's current state.
- Evidence: support-log.md line 54 (Turn 5, walkthrough 3 / bound-breaker 3 bullet).

No other disagreements:
- D3's "(D7)" pointer, the D7 criteria ("until the resume") and D5's restarts all agree with the changed D7 text.
- The D6 sentence and the D5 criterion agree with each other.
- The Round 2 support-log entry matches the edits.

Tool calls:
- Grepped `internal/fabricengine` for upstream-setting push flags, to see whether D5's "the pusher's push sets the branch's upstream" now also covers the code branch. My context had none of that package's push code. No push there sets an upstream, and D5 states it as new behaviour for any branch, so this is not a consistency finding.
