```yaml
lens: goal-scope
outcome: reported
```

Task: check the decision record against its board entry (goal-scope lens).

**Findings**

1. `decision` / MEDIUM — Open risks, D2: the entry wants `Drift-Verify` "after Webster and the Code perch, before Describe and Parent-Align". Since `fc75aef79` (2026-10-24), `Parent-Align` runs before the Code perch, so both cannot hold.
   - The record picks "after the Code perch, before Describe" and lets the agent accept drift that traces to merged parent commits (D7).
   - Question for the parent: should `Drift-Verify` compare after the Code perch, with parent commits inside the compared tree, or right after Webster, before `Parent-Align`, which leaves the Code perch's fixes uncompared? And is drift from parent commits acceptable by default?
   - Evidence: the board entry's Design item 3, and the loom recipe's `Webster` → `Parent-Align` → `Code-Bouncer` → `Describe` edges.

2. MEDIUM — D4 and D5, the trust check: the entry says to trust a baseline by `git diff &lt;commit&gt; -- '*.cs'` being empty, "not by the dirty flag". The record does the opposite:
   - D4 refuses a capture on any dirty tracked file.
   - D5 requires HEAD's whole tree to equal the capture's tree.
   - Issue #609 gives the reason for the entry's rule: task-state files change during discussion and plan.
   - If `_lyx/drift/scope.json`, `_lyx/drift/baseline.md` (written just before the capture) or any later record commit lands in the code branch's tree, every capture refuses or every first await halts.
   - Fix: restrict both checks to code paths, meaning the tree outside `_lyx/` and `.lyx/` or a hub-declared glob, or state why the whole tree is safe.
   - Evidence: the entry's Models section, the "Before reusing a baseline" bullet.

3. LOW — D2, `Drift-Await` gating: operator decision 3 says the rows are skipped like the PR rows are today when PRs are off. The record leaves `Drift-Await` ungated, so its history shows Done rather than Skipped.
   - The rationale is stated, but the record never names this as a departure from an operator decision.
   - Fix: say so explicitly in D2.

4. LOW — D2 and D6, escalation: the entry says the agent "escalates to the orch when unsure or past its budget". The record routes the Stuck through the shed stop chain, which is rule table, tackler, arbiter, then parent.
   - So the arbiter may settle the halt before the orch ever sees it, yet D2 says the chain "ends at the orch".
   - Fix: say the arbiter comes first, and what it may do at a `Drift-Verify` halt, such as `lyx drift accept`.

5. LOW — D4, refresh: the entry says "a refresh is an explicit operator command that backs up the old directory first". The record has no refresh verb.
   - Its stand-in is the automatic backup when `Drift-Baseline` captures again after a goto, which only works before Webster.
   - Fix: state that the goto is the refresh route.

6. LOW — D1, `verify_max_runs: 8`: the entry's Models experience asks for "many varied test runs", and 8 may starve the agent.
   - The value is an auto-pick and configurable, so it is no blocker.

**Goal points of the entry**

- Not a test tier; own config block with the flag, the capture and compare commands, the skip list and the agent models; commands take a gate slot: met (D1, D4).
- Discussion knows nothing of regression: met (Scope, Out).
- `Drift-Baseline` after Discussion, a Sonnet agent, scope off, partial or full, expected drift written down, capture in the background: met (D3, D4).
- `Drift-Await` mechanical; waits; a failed or timed-out capture halts; no later capture: met (D5).
- `Drift-Verify`: mechanical compare, no LLM when it passes; one strong agent with a budget that judges and fixes, writes its reasoning down and escalates; no perch: met, with findings 1 and 4.
- Darn wiring: `Drift-Baseline` before Darn-Write reading the board entry, capture run to the end, no `Drift-Await`, `Drift-Verify` before Code-Review: met (D2, D3).
- Operator decisions:
  - three new producers, no existing producer changed: met.
  - one key, off by default, this hub keeps it off: met.
  - rows skipped with no overlay: met for two rows, partly for `Drift-Await` (finding 3).
  - no review of `Drift-Verify`'s fixes: met.
  - no drift report type: met.
  - Models work follows on OSL-1046: met.
- Models harness facts: capture, compare and side capture are met. The trust check departs from the entry (finding 2). The never-overwrite rule is met, but there is no refresh verb (finding 5).
- No work belonging to another entry, and no deferral within scope. The `drift-verify` test gate is unlicensed but small, and the record lists it as an auto-pick.

**Decision notes:** `lyx board list` shows no `Decision for regression-baseline-verify:` note, and the record has no `Added after Discussion` entries, so there was nothing to read and nothing I could not reach.

**Tool calls:** none. My context already held `lyx board list`, `lyx board get` for this task and issue #609.
