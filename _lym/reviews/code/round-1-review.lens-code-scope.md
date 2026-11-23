```yaml
lens: code-scope
outcome: reported
```

Task: trace each change in the diff to the decision that asks for it (code-scope lens).

**Decisions**
- G-E1, from the Goal's Expected line: an amendment during Webster keeps the record of committed batches. Built by C1, through the state-record byte-equality assertion.
- G-E2, from the Goal's Expected line: the plan rework sees which cards already landed, so it neither re-creates their files nor needs a hand ruling. Built by C1, through the plan-gate assertion: a pass with no `create-already-exists`.
- G-Seen and G-Workaround describe the incident and the manual ruling. They ask for no change of their own, and A1 states how the incident is handled.
- A1, "Untraceable incident driven in a test": built by C1, which drives `decisionVerb` with `--amends-plan`, then a goto to Plan-Burler-Review, then the Plan-Burler-Fix plan gate.
- A2, "No test seam: the reported path loses no run record": built by C1. The test lands as a guard with no production change, and the range touches no production file, which matches "lands as a guard, not as a fix".

**Changes**
- C1, `internal/loomcli/amendduringwebster_test.go`, the whole new file across commits 7d353223c, 7da9d99a7 and 212e1f62a. It traces to A1, A2, G-E1 and G-E2. The two later commits only reword the file header, which is an edit C1 forces.

The range touches no other file, so the diff has no change outside the decisions, and every decision is built by C1.

**Findings**
None.

Tool calls: none. The inherited context already held the diff, the decision record, the commit log and `git diff --stat` for the range.
