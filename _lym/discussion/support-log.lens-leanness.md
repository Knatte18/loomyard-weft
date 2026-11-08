```yaml
lens: leanness
outcome: reported
```

Task: review the decision record and support log through the leanness lens, looking for content to move or cut.

**Findings**

1. MEDIUM — `## Auto-mode assumptions`: most bullets repeat decisions.
   Evidence: "maps live machine-local" repeats D4. "rebuild runs in the prime parent's dobby … plus by hand" repeats D5 and D6. "runs on `webster`, `darn-write` and `code-burler-fix` … cap of 40" repeats D7.
   Only two bullets carry anything new: the full design versus a staged one, and the prime worktree being clean and on the prime branch.
   Fix: keep those two and cut the rest. The support log's question ledger already records the picks.

2. MEDIUM — `## Constraints`: several bullets repeat a decision's own text.
   Evidence:
   - `PATTERN-lym-files` repeats D4 and D5, which already say the paths are named by lymdirs and the lock package.
   - `PATTERN-tolerant-records` repeats D4, which already says `internal/state` and unknown keys.
   - `PATTERN-child-processes` repeats D5 and D6, which already say "logs … waits", `selfexec.LymBinary` and "logged, and waited on".
   - `PATTERN-stencil-ownership` repeats D8, which already says Go-filled.
   - "Block coverage is evidence, never a verdict" repeats Goal and Scope Out.

   Keep the bullets with new content: the process-wide coverage counters, no test launching a test binary, `PATTERN-gate-slot`, `PATTERN-config-layers` (no reinterpretation), `PATTERN-cli-cobra` and `PATTERN-code-conventions`.
   Under the completeness-before-leanness test, the repeated bullets carry nothing the decisions don't already state.

3. LOW — `## Scope` "In" restates D1–D9 one to one, and `## Goal`'s middle sentences restate the same parts again.
   Fix: let Goal state the outcome and the "no part fails, refuses or deletes" bound. Keep Scope In as short pointers, or cut what repeats the decisions.

4. LOW — Scope "Out": "Changing the board entries `fabric-test-cuts` or `gate-shard-slow-package`" excludes something no one proposed. Cut it.

5. LOW — D4 rationale: "tracking them would churn the history" argues against a rejected alternative. That argument already sits in the support log's `## Rejected alternatives` ("Tracked maps: … churn history").
   Fix: keep only the positive reason, that each machine builds its own.

6. LOW — `## Open risks`:
   - The second bullet, about the cost of a first map, is a known cost, not a risk. It also pins a perishable figure ("about four minutes"). Move the point into D5's bound without the number, or cut it.
   - The first bullet ends "Overlap with its edits is textual only", which is no risk. The bullet reduces to a parent-facing note that D2 keeps the module-wide column, so it could be cut.

7. LOW — Emphasis, not exclusion: D7's rationale says judging against the map "keeps the cost to those tests' own run time". The step also builds a coverage binary for each package. Reword so the claim stays true.

8. LOW — Emphasis: the `## PATTERN changes` text "it pins behavior no test covers, and the gate's redundancy report flags it" reads as the report flagging a test that pins unique behavior, which inverts the meaning.
   Suggested wording: "it pins behavior no other test covers, which the gate's redundancy report checks". The length is within the 280-character cap either way.

I made no further tool calls.
