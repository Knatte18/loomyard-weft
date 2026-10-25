```yaml
lens: goal-scope
outcome: reported
```

Lens: goal-scope. I checked the record against board entry `regression-baseline-verify` (T1) and its parent decisions.

**Goal points of the entry:**
1. Regression is no test tier; it has its own config block with a flag, the capture and compare commands, a skip interface and the agents' models (entry, Design bullet 1). Met by D1.
2. Its commands take a gate slot. Met by D4 and D6; the record says "within the hub's slot count", a reading support-log bound-breaker 10 records.
3. Discussion knows nothing of regression and its stencil stays the same. Met by Scope Out.
4. Each Drift row is a no-show when the flag is off. Partly met: the Added-after-Discussion F10 ruling gates `Drift-Await`, but D2, the acceptance criteria and the support log still say the opposite (finding 1).
5. `Drift-Baseline` after Discussion: a Sonnet agent picks the scope (off/partial/full) and the expected drift, then captures in the background in the task worktree while Plan runs. Met by D3 and D4.
6. `Drift-Await` before Webster: mechanical, waits for the capture, halts on a failed or timed-out capture and never falls back to a later capture. Met by D5. `lyx drift defer` is a way forward after the halt, not a fallback capture.
7. No capture in a separate checkout. Met by Constraints and Rejected alternatives.
8. `Drift-Verify` after the Code perch, before Describe: the entry's "before Parent-Align" is overruled by F9. A mechanical compare, no LLM on a pass, one strong agent on a fail with a budget, which writes its reasoning and escalates when unsure or past budget. Met by D6. Escalation through the arbiter first is stated in D2, as the shed stop chain requires.
9. One agent judges and fixes and is no perch; a review, if added, is a single solo agent. Met by D6, which adds a test gate only and no review.
10. Darn: `Drift-Baseline` before Darn-Write reads the board entry and captures to the end; no `Drift-Await`; `Drift-Verify` after Darn-Write, before Code-Review. Met by D2 and D3.
11. Operator decisions: three new producers, no existing producer changed, one clearly named key off by default, this hub keeps it off. Met by D1 and Scope.
12. No drift report format in `contracts/specs/`. Met by Scope Out.
13. Models lives on OSL-1046, and the task builds and tests everything without Models. Met by Scope Out and the last acceptance criterion; the Models-side facts in Open risks are ones the operator placed off this machine.
14. Issue #609 harness facts: a refresh only by an explicit operator command with a backup, and a baseline never deleted or overwritten. Met by D4 and D7. The `*.cs`-diff reuse check is replaced by a stricter branch-point check (Rejected alternatives).

**Decision notes:** `lyx board list` holds no note titled `Decision for regression-baseline-verify:`, so none were read and none were unreachable. The parent decisions are the record's two Added-after-Discussion entries, F9 and F10.

**Findings:**

1. **BLOCKING · consistency · `decision-record.md` D2, Acceptance criteria, `support-log.md`.** The record still states the ungated `Drift-Await` that parent ruling F10 overturned.
   - D2 (decision-record.md:72-73) says `Drift-Await` "carries no entry gate" and shows Done with the flag off.
   - The first acceptance criterion (decision-record.md:229) pins "a loom run records `Drift-Await` as Done".
   - Both contradict the Added-after-Discussion entry at decision-record.md:199-203, which gives `Drift-Await` `entry_gate: drift.enabled` and has it record Skipped.
   - The support log keeps the overturned choice as current: Rejected alternatives (support-log.md:317) and Q6 (support-log.md:341).
   - D2's reason for staying ungated (a capture still running when the flag turns off holds Webster back) no longer holds. The record must say what happens then: Webster may build beside a running capture.
   - Fix: rewrite D2's paragraph and the acceptance criterion to the gated, Skipped behaviour. State the flag-off-mid-capture outcome. Drop or reword the rejected-alternative line and Q6.
2. **MEDIUM · consistency · `decision-record.md` Open risks, first bullet.** The bullet about `Drift-Verify`'s position against `Parent-Align` asks the parent a question the F9 entry already answers.
   - The bullet is at decision-record.md:222-223; the F9 entry is at decision-record.md:193-197.
   - The rubric makes such a line a finding whose fix drops it.
   - Q1 (support-log.md:336) still says the question is "raised to the parent under Open risks".
   - Fix: drop the Open risks bullet and note in Q1 that F9 settled it.

Tool calls:
- `lyx board list` filtered by the slug and "Decision for": to find this task's decision notes, which my context lacked.
