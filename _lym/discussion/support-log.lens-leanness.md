```yaml
lens: leanness
outcome: reported
```

Task: leanness-lens review of `decision-record.md` and `support-log.md`, looking for content to move or cut.

1. **LOW — Scope "Out" and the second Open-risks bullet.** Both state the same thing: the `Was ide-prime-setup` titles, folderOpen chain and `launch.json` are already handled, citing `b0e9f535` and `6b037083`.
   Evidence: Scope Out's first bullet, and the second Open-risks bullet.
   Fix: keep the commit citations in one place only. The Open-risks bullet can point at Scope instead of repeating them. Nothing the plan writer needs is lost.

2. **LOW — D7's rationale, Scope Out and the first Open-risks bullet.** All three say that area grouping moves to the viewer's filter.
   Evidence: D7's last rationale sentence ("Area grouping in the README is replaced by the viewer's area filter…"), Scope Out's "Grouping the README by area… see D7", and Open risks ("leaves area grouping to the viewer").
   Fix: state it once, in D7, and drop the D7 sentence or the Scope line.

3. **LOW — Auto-mode assumptions.** This section repeats picks the Decisions already state in full: the task labels and keys (D6), Notes and Done folding closed (D7), and grouping by project and section (D4). It also carries the reason for each pick, which the ledger's Q3 and Q7 already hold.
   Evidence: assumption bullets 1, 3, 4 and 5.
   Fix: the section is required, so keep each bullet to the bare self-made pick with its D-id. Move the "since the README moved to layers after the demo" reason to the ledger, where Q3 already has it.

4. **LOW — D4's rationale.** "This is the demo shown to the operator on 2026-10-04, regrouped…" is history, and the plan writer gains nothing from it. Auto-mode assumption 1 and Q3 already cover the regrouping.
   Fix: cut it, or move it to the support log.

5. **LOW — D6's "Mill holds `alt+shift+1` to `alt+shift+6`, which these avoid."** This is the reason the keys were picked, not a requirement.
   Completeness test: it does carry one constraint, that lym's keys must not collide with mill's block. Keep that as a short Constraint line, or drop it, since the keys are already fixed. Move the narrative to the ledger's Q7.

6. **LOW — Constraints restate the decisions.** The `PATTERN-cwd-resolution` bullet repeats D5's rationale ("lym resolves the hub path, so nothing is linked…"), and the `PATTERN-child-processes` bullet repeats D5's third sub-bullet.
   Fix: keep each in one place. The Constraints line can be the PATTERN pointer alone, with D5's rationale sentence dropped.

7. **LOW — the Goal bullets mirror Scope "In".** The three Goal bullets cover the same items as the first, fourth and fifth Scope-In bullets.
   Fix: shorten the Goal to its first sentence.

Outside this lens, noted for emphasis only:

- **MEDIUM — process.** Only this one lens fork appears to have been spawned. The prompt requires all six lenses in one message (interactions, existing-paths, bound-breaker, walkthrough, board-scope, leanness). The writer should check this.
- **LOW — D3.** D3 says "The owner-set write path commits the rendered names `Render` returns". The only basis is the exploration subagent's summary of `renderOwnerWrite`, which the writer never read. The record states it as a fact.

Further tool calls: none. My context already held the record, the support log, the rubric and the exploration results.
