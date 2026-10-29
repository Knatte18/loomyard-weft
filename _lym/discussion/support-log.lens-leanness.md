```yaml
lens: leanness
outcome: reported
```

Leanness lens: what in the decision record should move or be cut.

Findings:

1. LOW — Decisions D1, D2, D6, D7, D9, D10, D11, D13: each "Rationale:" paragraph partly argues against options that were not chosen. Examples: D10 says "`Awaiting` would halt the run…", and D7 says "Starting it per entry rather than per wake…". The spec says Decisions hold Decision plus Rationale only, with rejected alternatives in the support log, and those exact alternatives are already listed in `## Rejected alternatives`. Fix: cut the clauses about rejected options and keep the positive reason. Completeness test: nothing the plan writer needs is lost.

2. LOW — Constraints: the bullets for `PATTERN-github-auth`, `PATTERN-parent-lock`, `PATTERN-config-layers` and `PATTERN-gate-slot` restate what D5, D9/D11, D1/D5/D7 and D8 already say, sometimes word for word. For example, "`pr_account` is CategoryCrossPair" appears both in D5 and in Constraints. Fix: keep each fact in one place, either the decision or a one-line pointer to the PATTERN id. Keep the self-approval constraint, which is a new fact.

3. LOW — Auto-mode assumptions: each bullet restates a pick already made in D2, D7, D8 or Constraints, and the support log's Question ledger records the same picks. Fix: shorten it to the picks the decisions do not state as picks, or cut lines that are verbatim duplicates. Only the default-`on` bullet adds something, and it overlaps the first Open risk.

4. LOW — Open risks: the first bullet ("Default `on` changes how every GitHub-hosted hub lands…") duplicates the last Auto-mode assumption. Fix: keep one of the two.

5. LOW — Notes for the plan writer: the first bullet (`doc.go` and `README.md` change) is a docs obligation the repo's CLAUDE.md already imposes. It is harmless as a hint and could go. The `shuttleengine/open.go` note is a real design gap: D7 needs an interactive session that starts with a prompt. That requirement belongs in D7 itself, not in an optional notes section the plan writer may treat as a hint. Fix: move that one sentence into D7.

6. MEDIUM — D11: the third bullet carries Finalize's whole sequence (push, merge, fast-forward, done mark, refusal halt, how resume behaves), and the second bullet narrates the already-landed mechanism through `ParentHoldsTaskWork`. The already-landed path already exists (landingshed `doc.go`, line 38). Fix: state it as a constraint, "a GitHub-merged pull request uses Finalize's existing already-landed path", and drop the explanation of how the merge-tree test recognises a squash.

7. LOW — D5: "Rationale: … `gh auth token --user` exists from gh 2.40.0 … the installed gh 2.103.3 lists …" is a research fact and belongs in Constraints with its evidence. Moving it would make Rationale only the reason for the choice.

8. LOW — Support log, Turn 1: it lists every file read. That duplicates `relevant.md`. Cut it to a single line.

Nothing else stands out. The record is long but mostly carries decisions the plan writer needs.

Tool calls: none; my context already held both files and the rubric.
