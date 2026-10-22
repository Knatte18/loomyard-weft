# Card 17 — Plan-Burler-Review answer key names the amendment

**Edit:**
- `contracts/recipes/loom-recipe.yaml`

**Intent:**
D7: the Plan-Burler-Review row's `fasit.instructions` gain the amendment rule, and the plan-conflict clause is reworded to the same editable-card rule.

The instruction gains: when `_lyx/loom/amendment.yaml` exists, each decision-record entry it names by title is a finding for each card that entry names while the entry's change is uncarried for that card, or, when the entry names no card, a finding until some card carries it.
An entry's change is carried for a named card when that card carries it, or, for a named card whose batch is done, when a card inserted with `lyx loom insert-card` after the last begun batch carries it.
A decision names each card by slug as well as number, because `insert-card` renumbers every later card, so the reviewer matches cards by slug.
Such a finding is fixed by editing only cards Webster's auto-rebaseline accepts, a pending card or a card of a failed or in-flight batch, or by inserting a card with `lyx loom insert-card` after the last begun batch.

The existing plan-conflict clause's "fixed by editing only cards whose batch has not begun" is reworded to the same rule, so the two clauses agree when both bind one round and a card of the failed batch that stopped on the defect stays editable.

The YAML stays one document whose row keys `shedbuild.Parse` accepts; the embedded recipe's parse tests keep passing, and no test asserts the instruction's prose.

**ImpactSummary:** prose of one row's reviewer instruction in the embedded loom recipe; no key, row or edge changes.
