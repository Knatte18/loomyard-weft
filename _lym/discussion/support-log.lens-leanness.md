```yaml
lens: leanness
outcome: reported
```

Task: review the decision record and support log through the leanness lens, looking for content to move or cut.

**Findings**

1. MEDIUM — `## Auto-mode assumptions` repeats decisions word for word. Each bullet restates something a decision already settles: `{beam}` as a path is in D4 and D8, role `agent` and the beam set are in D8, `follow` first is in D1, and edits reaching a session at the next attach is in D5. Evidence: compare each bullet with D1, D4, D5 and D8. Fix: shorten each bullet to the pick plus "(D4)" or the right decision id, and keep the rationale in one place only. The section itself is required, so keep it.

2. LOW — D3's rationale is mostly deliberation that belongs in the support log. Its account of why the per-worktree reed local layer cannot hold the key repeats the support log's Rejected alternatives. Completeness-before-leanness: keep the claim the plan writer needs, that the buttons fit `CategoryMachine` as a fact of the machine's desktop so no category or PATTERN entry changes. Move the rest. Evidence: D3 Rationale's first two sentences against the support log's first rejected alternative.

3. LOW — In `## Constraints`, the `PATTERN-wait-mark-display-only` bullet adds nothing. It states that nothing changes and imposes no constraint, and D5 already says which new options are display only and which one decisions read. Fix: cut it.

4. LOW — In `## Constraints`, the `PATTERN-shell-guard-table` and `PATTERN-spawn-observability` bullets repeat D6 and D7, which already say values are quoted through `internal/shell` and that the spawn, exit and output are logged at Info. Fix: keep them only as pattern pointers, or drop them. Low value either way.

5. LOW, consistency — The `## Open risks` line is a design bound, not an open risk. Whether a tracked `reed.yaml` button may run as the external caller is a call the writer can make, and has made. It is not a question for the parent, so as it stands it will be raised to the parent for no reason. Fix: fold it into D7's Bound paragraph and leave Open risks with "none".

6. LOW — The D8 acceptance bullet is vague. "each command runs from where the button runs it" cannot be checked. Fix: name the one check that matters, that CANCEL in a task session runs `lym batten cancel` from the beam's worktree and is not refused, or cut the clause.

7. LOW — In `## Notes for the plan writer`, the second bullet partly restates the test-tier rule in `PATTERN-test-speed`. It is a harmless hint and optional by contract, so cutting it is the writer's call.

8. LOW, emphasis — The rationale in D5 cites tmux `status.c` (expand, then draw). That is the evidence for a fact the design rests on, so it is correctly kept in the record. Do not move it.

Nothing in the record belongs out of Discussion altogether. The Scope, Decisions and Acceptance criteria carry no rejected alternatives and no narrative beyond the points above.

Tool calls made: none. My context already held both files, the rubric and the spec.
