# Plan round 2 — fixer report

## Changed

- **F1** (MEDIUM, scope) — fixed differently in one detail.
  The yamlengine rule now keys on the text being set as well as on the leaf: a scalar whose text is a spelled null, or one set on a `!!null` leaf, takes the tag its text resolves to.
  `SetValues` tests the working-tree leaf at the point `.Value` is assigned, after `applyExistingOverrides` has copied the existing tag; `SetSparse` applies it for both directions.
  `TestSetValues` gains the `plan_max_bounces=~` over an existing `plan_max_bounces: 2` row, and the `TestSetSparse` row sets both `2` and `~` over the `~` template leaf.
  The rule lives in the new card 3 (F2).
  Not made as proposed: the `fix` key's "a value whose own text resolves to null" would also catch an empty value, since YAML resolves an empty plain scalar to null; that would turn `--set <key>=` on a string leaf from an empty string into a null, which nothing asked for.
  The card limits the rule to a spelled null (`~`, `null`, `Null`, `NULL`) and says an empty value keeps its leaf's tag.
  The old sentence "No template held a null leaf before this card, so no existing set changes" was false under the widened rule and is replaced by the behaviour change it now makes (a spelled-null set on a non-null leaf writes a null).
- **F2** (LOW, design) — fixed as proposed.
  `lyx loom insert-card 3 yamlengine-null-leaf-set` added `03-yamlengine-null-leaf-set.md` with the four yamlengine Edit targets, the rule, the test rows, an ImpactSummary and Commit `3:`.
  Card 4 (was 3) lost those targets, its yamlengine bullet, its `TestSetValues`/`TestSetSparse` sentence and the yamlengine mention in its ImpactSummary, and its Intent now says its null budgets rely on card 3.
  Renumbered prose mentions: card 1 "cards 4 and 5"; card 2's three "card 4"; card 4 "card 5 opens darn's local layer"; card 5 "since card 4"; card 6 "card 7 turns either into a refusal"; card 7 "from card 9, batten's hold"; card 8's two "card 9". insert-card had already rewritten the Commit prefixes and the Card Index.
  `lyx loom validate-plan` reports `ok: true`, with informational create-new-unit findings only.
- **F3** (LOW, design) — fixed as proposed.
  Card 1's `CheckLocal` paragraph passes a nil only for `SoftKey(p)`, and refuses a nil `RunSoftKey`, `RunHardKey` or per-perch hard key with `must be an integer`; `TestCheckLocal` refuses a null `review_max_bounces`.
- **F4** (LOW, consistency) — fixed as proposed.
  Card 8's (was 7) last `TestHoldReading` row names `round-1-circling-decision.md` in Plan-Bouncer's run directory with `round: 1` and `decision: continue`, checked against `circlingDecisionPath` and `readCirclingFile` in `internal/shedadapters`.

## Disputed

None.

## Deferred

None.
