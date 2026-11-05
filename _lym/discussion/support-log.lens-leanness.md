```yaml
lens: leanness
outcome: reported
```

Task: leanness lens review of the raddle decision record and support log.

**Findings**

1. **LOW — `## Auto-mode assumptions`: it repeats decisions the record already states.**
   Evidence: each bullet restates a decision. The approval verb run on the operator's word is in D7. Released tasks and the `tasks` shape marking the entry done are in D9. The `opus[high]` default is in D4, and `tapestry`/`darn` only is in D5 and D6.
   Fix: shorten each bullet to a pointer ("D9: filed released; auto-pick, see ledger Q9"), or keep only the fact that these were self-picks. The support log's question ledger already records the reasons.

2. **LOW — `## Constraints`: several PATTERN bullets only restate the entry.**
   Evidence: these bullets say what the PATTERN line says and add nothing specific to this design:
   - `PATTERN-refusal-way-forward`: "every new refusal names its way forward"
   - `PATTERN-agent-messaging`: "go through creel"
   - `PATTERN-tolerant-records`
   - `PATTERN-dobby-starts-runs`

   The plan writer already gets the full PATTERN list.
   Fix: keep the bullets that tie an entry to a named artifact, such as sole-parsers → `internal/raddleparser`, durable-vs-ephemeral → `_lym/usage/` and `_lym/raddle/`, and loom-contract → the approve verb. Cut the restating ones, or merge them into one line.

3. **LOW — `## Open risks`: the `pawl-orch-landing` bullet is not a risk.**
   Evidence: the bullet itself says the two tasks "meet only in `contracts/recipes/pawl-recipe.yaml`". That is a text overlap in one file, which the board-scope rule says is no finding.
   Fix: move it to the support log's Interview turn 1, which already names that peer, or drop it.

4. **LOW — `## Scope` "In" list: it repeats the D1–D15 headings in other words.**
   Evidence: each bullet maps one-to-one onto a decision. "The approval verb, the board re-check, the auto-approve key" is D7 and D8, for example.
   Fix: keep "Out", which carries information found nowhere else. Shorten "In" to one line, "D1–D15", or a short grouping.

5. **LOW — `## Constraints` Fact 2: part of it is narrative that belongs in the support log.**
   Evidence: "the token audit in `docs/research/token-audit-2026-10-22/` was computed by hand from transcripts" is the history of how the gap was found.
   Fix: keep the clause the design rests on, "no lym code records a run's token cost; only review rounds record a token reading", and move the audit history to the support log, which already holds it in turn 1. This is the completeness-before-leanness test: the requirement stays and the history moves.

6. **MEDIUM (a point to stress, not a cut) — D12–D14 are a cost subsystem as large as the raddle recipe itself.**
   Evidence: they add per-session usage records, changes to the shuttle engines, Finalize recording diff size, and two new report verbs.
   The entry does ask for measured costs from the hub's own run records, so they belong in scope. But the record never says plainly that "`lym report timing` and the token audits" do not exist as hub run records. It only says so in a Constraints fact.
   Fix: put one sentence in D12's rationale that the entry's named sources hold no token cost in a hub, so recording it is part of this task. That stops a reviewer from reading D12–D14 as gold-plating.

7. **LOW — D3: a short phrase would cover the darn behaviour change.**
   Evidence: the rationale "raddle's part tasks are filed discussed (D9), and a darn part must build from its part record" is enough. The added clause "which closes the open line of note `discussed-task-split`" repeats Open risks bullet 3.
   Fix: state it once, in the decision. The Open risks bullet is then the place for the consequence alone: existing discussed darn entries change behaviour.

8. **LOW — D4: the last sentence holds stencil wording.**
   Evidence: "it states that a tapestry carries a large fixed start cost…; the figures come only from `lym report cost`". The `Review:` D4 criterion already checks this.
   It can stay, because it carries a requirement the plan writer needs, so this is no move. Note only that the requirement is stated twice.

No BLOCKING finding: nothing in the decision record is deliberation that belongs only in the support log, and no section carries content outside Discussion's remit.

Tool calls: none. My context already held both files, the rubric and the exploration.
