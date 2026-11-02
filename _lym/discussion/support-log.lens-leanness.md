```yaml
lens: leanness
outcome: reported
```

Task: review the decision record and support log for content that should be moved or cut (leanness lens).

- **MEDIUM — D2, Rationale.** A rejected alternative sits under `## Decisions`. "Returning pending was weighed and rejected, see the support log: re-evaluation restarts from the first entry…" breaks the compaction rule that `## Decisions` carries only the decision and its rationale. The plan writer still needs the fact behind it, and Constraints already holds it ("Fact: no shipped gate closure returns `Pending` today, and a gate evaluation runs every entry from the first… this is why D2 blocks instead"). The support log's Rejected alternatives holds the deliberation. Cut the sentence from D2 and lose nothing.
- **LOW — D5, Rationale.** "A Go-side table of finding codes would duplicate each spec's own list and go stale" is also a rejected alternative, and the support log already holds it. Keep the first clause, that reports name each failure and its place, and drop the rejected-table half.
- **LOW — D3, Rationale, and Constraints.** The fact that config merges a mapping entry key by key is stated twice: in D3's rationale with its source, and again as the last Constraints fact. Keep one copy. The Constraints copy is the better home because it already groups the facts.
- **LOW — D4 opening, and Constraints.** The fact that the card gate is the `card` gate key with `--card <id>`, rendered by `cardgate.go`, appears both in D4's first sentence and in the last Constraints fact. Keep one copy.
- **LOW — Open risks, and D2.** The pause risk is stated twice. D2 says "While the closure blocks, the loop reads no pause, cover or liveness signal", and Open risks repeats it with its effect. Keep the bound in D2 and the effect in Open risks, or merge them, but don't repeat the mechanism.
- **LOW — Scope, Out.** The last bullet ("The shuttle gate loop … no change") repeats D2's "no `shuttleengine` change is made". The first Out bullet also carries a design hint ("A row gains a fixer later by adding the `fixer` key…, with no code change") that D3 already implies. Both can go.
- **LOW — Constraints, the PATTERN compliance list.** This roughly ten-line list says which entries the design meets without changing any of them. It fails the cut test: the plan writer applies PATTERN from the overview anyway. It is deliberation evidence that belongs in the support log. The only decision-relevant PATTERN content is the `## PATTERN changes` bullet. Before moving it, keep any item that sets a requirement the decisions don't already state; `PATTERN-keep-warm-cap`'s point about the card caller's wait already lives in D4.
- **LOW — Goal, D1 and D6.** "Generalised from webster's cold verify-gate fixer… which becomes a caller of the same core" is said in the Goal, again in D1 ("lifted from… not written beside them") and in D6's rationale ("the board asks for generalisation rather than a second fixer beside webster's"). Keep the Goal sentence and D6's mechanics, and trim the two restatements.
- **LOW — Scope, In.** The bullet "Tests for the above" adds nothing, because Acceptance criteria carries the scenarios.

Nothing I found should be excluded outright; every item above is duplication or deliberation narrative. The Notes for the plan writer are acceptable hints.

Further tool calls: none.
