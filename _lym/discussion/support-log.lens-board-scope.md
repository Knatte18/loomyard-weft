```yaml
lens: board-scope
outcome: reported
```

Lens: board-scope. I checked the record against the board entry, the touching entries, the peers and the decision notes.

**Findings**

1. **MEDIUM** — D3, ask keys (class `design`). D3 leaves `code_review_ask_parent` out, but the entry says the setting "sets the existing ask keys".
   - Evidence: the entry's "Open for the design" says the setting "sets the existing ask keys and the new halt routing".
   - Its "What exists" names exactly two keys, `code_review_ask_parent` and `judge_ask_parent`.
   - D3 forces `judge_ask_parent` and `pr_review_ask_parent` but leaves `code_review_ask_parent` to its own value.
   - The support log's rejected alternatives explain that choice, but the record departs from the entry's wording without saying so.
   - Fix: force `code_review_ask_parent` on under autonomy, or name the departure in D3 as a deliberate reading of the entry.

2. **MEDIUM** — Scope "Out", the rollout (class `decision`). The record defers the entry's "Wanted first in the Models hub on OSL-1046 and in loomyard" to an operator config write after landing.
   - Question for the parent: should the run turn `autonomy` on in loomyard's own hub as part of its landing? Or is that rollout the operator's call after landing, as the record assumes?
   - Turning it on in the Models hub is another hub's config, which this run cannot write.
   - A related open fact: does the Models hub on OSL-1046 use a layout in which the default and worktree layers of `beam.yaml` resolve as D1 assumes?

3. **LOW** — D7, goal case 2 (class `consistency`). The entry says "lym opens the beam's pull request ready for merge, then notifies the operator". In D7 the orch opens it through `lym beam pr open`, and only after a `[beam drained]` line.
   - D7's rationale explains this through PATTERN-github-auth and the orch writing the description, so it is not a contradiction.
   - But the PR is opened ready for review, and the notice may still name blockers. "Ready for merge" is met only when `lym beam pr status` shows none.
   - Fix: say in D7 whether the operator is notified while blockers stand, or only once the status is mergeable.

**Goal points**
- The parent orch receives and settles every halt of every member run: met (D5).
- Autonomy decided per beam, one beam autonomous beside one that is not: met (D1, worktree layer).
- Operator case 1, a severe halt the orch cannot settle: met (D5's closed list and the `[operator] severe halt` relay).
- Operator case 2, a merge-ready beam with the PR opened and the operator notified: met through the orch's verb (D6, D7, D8), with finding 3.
- Wanted first in the Models hub and in loomyard: deferred (finding 2).
- Which halts the orch may settle, and how severe is marked: met (D5).
- Member PR review off or reviewed by the parent: met, reviewed by the parent (D4).
- A per-beam switch, plus hub and prime root tasks: met (D1, default and prime worktree layers).
- It sets the existing ask keys: partly met (finding 1).
- A drift does-not-match still stops the run: addressed (D3 judge ask, and D5's settling by the orch).

**Entries, peers and parent commits checked**
- `harden-fabricengine` (peer, fabric only): no conflict.
- No parent commits: `lym loom peers` lists none.
- `borrowed-orchestrator-ideas` (P2): no conflict.
- `orchestrator-survey-ideas` (F17): no conflict. Its review lead mentions the ask keys only as an extra judge, not a quorum, and the record does not touch that.
- `move-task-by-handback` (F3), `orch-launch-refused-by-gate-key` (B1) and `stale-records-ref-blocks-rerun` (B2): no conflict, judged from the board list text already in context.

**Decision notes**
- None exist: the board list holds no `Decision for orch-full-autonomy-mode:` note, and there was none I could not reach.

**Tool calls**
- One Bash call: a fresh `lym board list` filtered for decision-note titles, to catch a note added since the writer's read, and `lym board get` for P2 and F17, whose bodies were not in context.
- `lym board get` for this task and `lym loom peers` were already in context from the writer's Step 1.
