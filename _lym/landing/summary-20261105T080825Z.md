# Seed-only hub config follows the template; light gets the lens fan back

A template change now reaches a hub's seed-only config files on the next reconcile, and a value the operator set by hand stays theirs.
Before this, the hub reconcile seeded `profiles.yaml` once and never touched it again, so a hub seeded before `6d92bff1` never got that commit's `code_reviewer_fan: ""` keys and nothing told its operator.

## What changes

- **Template base.** Beside each seed-only board file the reconcile keeps `<module>.template-base` (`configengine.TemplateBaseFile`): the template the file last followed, under a header saying not to edit it.
  It is committed on Bolt with the file, by the walk, by `lym config reconcile --layer default --apply`, by `lym fabric reconcile` and by clone.
  The seed-only modules are `profiles` (`profiles.yaml`, hub-wide) and `models` (`models.yaml`, the per-worktree repo default on the board).
- **Three-way merge.** `yamlengine.FollowTemplate` merges key by key, descending into mappings and comparing a scalar or a list whole:
  - a key the hub file holds as the base does takes the template's value, and is reported under `added`, `updated` or `removed`;
  - a key the operator set, added or removed stays as it is, and is reported under `differs` with both values when the template holds a third value;
  - a hub file with no base yet, which today is every hub, gains each key it lacks and keeps every value it holds, reporting under `differs` a value the template holds otherwise.
    That first pass writes the base, so later template changes follow the rules above.
- **Reporting.** `configsync.Result` gains `Updated`, `Differs` and `WroteTemplateBase`.
  `lym config reconcile --layer default` lists `updated` and `differs` in its dry run beside `added` and `removed`.
  After a walk's board commit lands, what each seed-only file took from the template and each operator value it kept go to the hub's orch as one `[reconcile notice]` line, through the notifier reinterpretations already use.
- **Template.** `light` drops `loom.code_reviewer_fan: ""` from its tapestry, darn, pawl and sampler blocks, so a light run's Code-Review round takes the hub's lens fan again; `docs` and `speed` keep their solo round.
  On the loomyard hub, with no base, the next walk adds `code_reviewer_fan: ""` to `docs` and `speed` and leaves `light` with the fan.
  On the Models hub, once its orch removes the key from `light` by hand, the hub file and the template are equal there.
- **Unsettled pairs.** The profiles `Lacking` hook is now `Unsettled` (`runprofile.UnsettledFormerBlocks`) and lists only a former-name recipe block beside a differing current one.
  A template block the hub lacks is no longer listed, since the merge now adds it or keeps the operator's removal.
  The result field and the envelope key are renamed to match (`unsettled`).

Docs follow the new rule: `internal/runprofile/doc.go`, `internal/hubreconcile/doc.go`, `internal/configcli/doc.go`, `internal/configreg`'s `SeedOnly` doc, `docs/overview.md` and both templates' header comments.

## Design decisions

- **No base: add every lacking key, whole profiles included.** The record's rule for a hub with no base treats a key the hub lacks as never chosen.
  A hub with no base whose operator deleted a starting profile, or a `models.yaml` alias, gets it back once on the first pass.
  The pass writes the base, so a second deletion stays deleted.
- **No base: a key only the hub holds is kept silently.** An operator's own profile or alias is not reported under `differs`, since seed-only files are open-ended by design and nothing tells such a key from one a template dropped.
- **Base content is the whole template.** The base records the template as of the last reconcile, also for keys the operator overrode, so `differs` fires only on a template move after the operator's change, not on every reconcile.
- **A key the template dropped is removed by every board reconcile, the walk included.** The record asks for that removal.
  `PATTERN-config-layers` says only `lym config reconcile` prunes unknown base-layer keys.
  Its PATTERN line and background now state that a seed-only key its template base holds is not unknown, and the background records the accepted downgrade flip-flop: an older binary drops a newer template's key, and the newer binary's next walk adds it back.
- **Orch notice.** The record asks only for `updated` and `differs` in the reconcile output.
  The walk also mails them to the orch, since the problem was that nothing told either hub's operator that the template had changed.
- **Template change kept apart from the merge.** The base path is a per-module file rather than a section inside `profiles.yaml`, so the operator-owned file holds only operator content, and the base uses the board config dir's no-`.yaml`-suffix convention so no module reader takes it for a module file.
- **Prime-copy seed.** A per-worktree seed-only default seeded from the prime's copy follows the template with no base, so it gains the template's keys the copy lacks.

## Tests

- `TestFollowTemplate` (`internal/yamlengine`) covers each merge case: update with comments kept, removal of a whole unit, removal that empties a mapping, a subtree or flow-mapping key added, an operator value kept with and without a template move, an operator removal kept, the no-base rules, a scalar replaced by a mapping, an empty file and a non-mapping document.
- `TestReconcileHubWideAt_SeedOnlyFileFollowsItsTemplate` (`internal/configsync`) drives `profiles.yaml` through the light-fan removal and the other cases above, checking that a dry run writes nothing, that an apply writes an absent or stale base, and that a second apply has nothing to do.
- `TestEnsure_CarriesATemplateChangeIntoASeedOnlyConfigFile` (`internal/hubreconcile`) runs a walk over a hub holding the older template with a tuned value.
  It checks that the light fan goes, the tuned value stays, the file and base land in the board commit and the orch notice names the removal.
- The `lym config reconcile --layer default` integration test now checks that the dry run lists the light-fan removal under `removed` and writes nothing.
- The existing profiles rows, `TestUnsettledFormerBlocks` and the template ladder test are updated to the new rule and template.
