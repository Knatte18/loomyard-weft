```yaml
lens: leanness
outcome: reported
```

Task: a leanness review of the decision record and support log, looking for content to move or cut.

**LOW: `## Auto-mode assumptions`, third bullet.** "`pr-review` keeps `diff`" says again what D1 decides and Q2 records. Evidence: D1 says "`pr-review` stays `diff`" and gives its own rationale. Cut the bullet. No requirement is lost.

**LOW: `## Auto-mode assumptions`, first bullet, and `## Open risks`, fourth bullet.** Both say tier 2 keeps the import scope, which D5's unmapped-tier miss case already rules. Keep the risk, since it is the gain caveat. Trim the assumption to its cost claim: "the cost the board names is tiers 0 and 1 across importing packages".

**LOW: `## Scope`, Out, last bullet.** "Changing this hub's own `gate.yaml` … (see Open risks)" repeats Open risks' second bullet. It is a scope exclusion, so keep one place. I suggest keeping the Out bullet as a bare exclusion and letting the risk carry the `lym config gate --set` detail. Or drop the Out bullet.

**LOW: D2, Rationale, second clause.** "`BuildBinary` … already takes a comma-separated coverpkg list, so the change is in what testmap passes and records" is an implementation hint, not a design reason. Move it to `## Notes for the plan writer`. The Go 1.26 coverpkg fact in Constraints already carries what the design rests on.

**LOW: D7, last sentence.** "The gate's result and failure report are otherwise unchanged" is padding. Nothing in the record suggests they change. Cut it.

**LOW: Constraints, PATTERN-test-speed bullet.** "selection logic tests run on fixture maps at tier 0, a testcover cross-package build at tier 1" is per-test tier assignment, which belongs to Plan. Keep only "each test takes the lowest tier that fits", or move the tier split to Notes for the plan writer.

**LOW: `## Scope`, In, first bullet. This is a consistency finding, outside leanness.** It says the keys that "today run tiers 0 to 2 per diff", and D1 includes `webster-fix`. The hub's `gate.yaml` runs `webster-fix` on tiers `[0, 1]`, and the template runs every key on `[0]`. Rephrase the bullet as "the diff-scoped Go gate keys", without the tier claim.

Nothing in Decisions re-argues an alternative, and the rejected options sit only in the support log. The Interview turns in the support log repeat exploration facts, which belongs in the support log, so there is no finding there.

Further tool calls: none.
