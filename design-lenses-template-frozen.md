# A hub lenses.yaml copies every template entry, so template edits never reach it

`lenses-template-frozen` · Task · enhancement, config, burler

Depends on: [`writer-reviewer-lens-align`](design-writer-reviewer-lens-align.md)

## Seen (orch, 2026-10-20)

After `lenses-module` landed, the hub's `lenses.yaml` on Bolt holds every template lens and fan word for word, because the migration and reconcile wrote the whole set into it.
Reconcile keeps any entry the file holds, prose and fan lists unchanged, so no later template edit to a lens or fan reaches the hub until the operator deletes or edits that entry by hand.
`writer-reviewer-lens-align` changes the template's `twill-plan`, and the darn writer pointed out that this hub keeps the old five-lens `twill-plan` as a result.

## Idea

The hub file holds only what the hub changed: an entry identical to the template's is left out, so a template edit reaches every hub that did not tune that name.
Reconcile could drop an entry that equals the template's current value, or the file could record which entries the operator tuned.
