# Decision record — lenses-template-frozen

## Goal

A base-layer file's per-name maps hold only the names the file changed, so a template edit to a lens or fan reaches every hub and repo default that did not tune that name.

## Scope

- `yamlengine.Reconcile` stops copying template names into a per-name map and drops each name whose value equals the template's, reporting each drop per name.
- `yamlengine.SetValues`, the base-layer `--set`, stops writing template names back into a per-name map, so a `--set` does not refreeze the template's entries.
- `configsync` carries the drops into `Result`, writes a file whose only change is a drop, and `lyx config reconcile` and the hub reconcile walk report them.
- The `lenses.yaml` header comment in `internal/lenses/template.yaml`, `internal/lenses/doc.go`, `internal/yamlengine/doc.go` and `docs/overview.md`'s `internal/lenses` row state the new behaviour.
- Not in scope: an entry equal to an older template value stays as a tuned entry; this hub's stale `twill-plan` is deleted by hand after landing.

## Decisions

### Equality is the decoded value

Decision: a file name equals the template's when both nodes decode to deeply equal Go values; style, comments and quoting do not count.
Rationale: the board defines equal as the same lens prose or the same ordered fan list, which is the decoded value, and a block scalar and a quoted string with the same text load identically.

### Reconcile writes the per-name map with the tuned and file-only names only

Decision: the merged mapping at a per-name path P holds, in template order, each template name the file holds with a value other than the template's, then each name only the file holds, in file order.
A template name the file holds with the template's value is left out and reported in a new `dropped` return as `P.<name>`, sorted.
A template name the file lacks is left out and no longer reported added.
Rationale: the board's goal; a name the file lacks already resolves from the template at load through `yamlengine.FillMissing`.
Bound: the drop removes only a value equal to the template's current value at the base layer, so the loaded value is unchanged; the sparse-layer `Prune` path never drops, since there a template-equal name can override a tuned value below it.

### An absent or null per-name map is written as an empty mapping and reported added as P

Decision: when the file lacks P or holds null there, the merged document holds P as an empty mapping and `added` reports `P` itself.
Rationale: a full base layer holds every template key, the load's fill refuses a null where the template holds a mapping, and reporting P makes the apply write the file.

### The base-layer --set drops template-equal names after its pairs apply

Decision: `SetValues`, after applying its pairs, removes from each per-name map every name whose value equals the template's, including a name a pair just set to the template's value.
Rationale: `SetValues` builds its output from the template, so without this a `--set` on any key rewrites every template lens and fan into the file and freezes them again; dropping a template-equal name leaves the loaded value unchanged.

### configsync reports drops as Dropped and writes on a drop

Decision: `configsync.Result` gains `Dropped`, filled at both base-layer reconcile sites, and a non-empty `Dropped` triggers the write on `--apply` like `Added` does; `lyx config reconcile`'s JSON carries it as `dropped` when non-empty, and `internal/hubreconcile`'s walk log carries it.
Rationale: the board asks for a per-name report, and a file whose only change is a drop must still be rewritten.

### The load log leaves out per-name names it fills

Decision: `configengine`'s "filled missing keys from template" Info log omits filled keys under a declared per-name map.
Rationale: after this change a base file lacking template names is the normal state, so logging them on every load is noise; a filled P itself is still logged.

## Constraints

- `PATTERN-config-layers`: only `configengine` reads and stacks layer files; the change stays in `yamlengine`, `configsync` and the load's log filter.
- `PATTERN-comment-line-breaks` and `scribe:prose`: one sentence per line in the template header, doc comments and docs.
- `PATTERN-test-economy`: new behaviour lands as rows in `TestReconcile`, the SetValues table and the configsync lenses tables, not new tests.
- Reconcile stays idempotent: a second reconcile of its own output adds, drops and removes nothing.
- Key moves and `Migrate` hooks still write what they move; the drop then applies to the moved file like any other.

## Auto-mode assumptions

- A darn run holds no interview; every call above is the writer's own pick from the board entry and the code.
- Extending the drop to `SetValues` is within the board's goal although its body names only reconcile, because otherwise any base `--set` reintroduces the frozen copies.

## Open risks

- Tests elsewhere that read a reconciled or `--set` lenses file and expect every template name in it fail and need their expectations changed.
- A hub that reconciles with an older binary after a newer one dropped its names gets them neither added nor frozen, since the older binary's reconcile re-adds them; the next newer reconcile drops them again.

<!-- lyx:carry-over Darn-Review -->
- Darn-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/darn/round-1-review.md; fixer report: _lyx/reviews/darn/round-1-fixer-report.md.
  - filled-keys-log-filter-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Darn-Review -->

## Acceptance criteria

- Reconcile drops a file name whose lens prose or fan list equals the template's and reports it in `dropped`; a block scalar and a quoted string with the same text count as equal.
- Reconcile keeps a name whose value differs from the template's, and a name only the file holds, and reports neither as dropped.
- Reconcile leaves a template name the file lacks out of the merged file and out of `added`.
- Reconcile writes an absent or null per-name map as an empty mapping and reports P added.
- A second reconcile of the merged output adds, drops and removes nothing.
- A file whose dropped names were its only template entries loads with every template lens and fan.
- `SetValues` on a base file leaves template names out and drops a name set to its template value, while keeping a tuned name.
- `configsync`'s hub-wide reconcile of a lenses file holding template-equal entries reports them in `Dropped`, rewrites the file without them on apply and writes nothing on a dry run.

## Notes for the plan writer

- `mergePerNameMaps` in `internal/yamlengine/reconcile.go` is where the merge and the drop sit; a shared value-equality helper serves `SetValues` too.
