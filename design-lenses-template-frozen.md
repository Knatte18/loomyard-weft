# A hub lenses.yaml keeps only what the hub tuned, so template edits reach it

`lenses-template-frozen` · Task · enhancement, config, burler · running · Merge-In

## Seen (orch, 2026-10-20)

After `lenses-module` landed, the hub's `lenses.yaml` on Bolt holds every template lens and fan word for word, because the migration and reconcile wrote the whole set into it.
Reconcile adds each template lens or fan the file lacks and keeps every entry the file holds, prose and fan lists unchanged.
So no later template edit to a lens or fan reaches a hub until the operator deletes or edits that entry by hand.
`writer-reviewer-lens-align` changes the template's `twill-plan`, and this hub keeps the old five-lens `twill-plan` as a result.

## Goal

A hub's `lenses.yaml` holds only what the hub changed, so a template edit reaches every hub that did not tune that name.

- Reconcile drops a lens or fan whose value equals the template's current value, and reports each drop per name.
  Equal means the same lens prose or the same ordered fan list.
- A dropped name resolves from the template at load, as a name the file lacks does today (`configengine.LoadOrTemplate`).
- Reconcile no longer copies a template lens or fan the file lacks into it, since the template supplies it at load.
- Reconcile still never overwrites a value the file holds, and its migrations (key moves such as `burler.lenses -> lenses.lenses`, renames, retired keys) still write what they move; dropping an entry equal to the template is the one removal added, and it leaves the loaded value unchanged.
- An entry that differs from the template, including one the template lacks, is kept as today.
- The change sits in the per-name map handling `lenses-module` added (`internal/yamlengine/reconcile.go` and `internal/configsync`, declared through `configengine.Spec.PerNameMaps`), so it applies to every per-name map, not only `lenses`.
- The `lenses.yaml` header comment in `internal/lenses/template.yaml`, which says reconcile adds every template entry and a later template edit reaches the file only once its entry is deleted, says the new behaviour, and `internal/lenses/doc.go` follows.

## Not in scope

An entry that equals an older template value is a tuned entry to reconcile and stays.
This hub's `twill-plan`, which equals the old five-lens template fan, is therefore deleted by hand by the orch after landing, not by this task.
