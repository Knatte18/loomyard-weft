# Seed-only hub config follows the template; light regains the lens fan

A template change now reaches a hub's seed-only config files on the next hub reconcile, and a value the operator set by hand stays theirs.
Before, `profiles.yaml` and `models.yaml` were seeded once and never touched again, so a hub seeded before a template change never learned of it.
The loomyard hub hit this: it missed `loom.code_reviewer_fan: ""` and kept running light darn reviews with the lens fan, while the Models hub, seeded later, ran them solo.

`light` also drops `loom.code_reviewer_fan: ""` from every recipe block of the `profiles` template, so a light run's first Code-Review round takes the hub's fan again.
`docs` and `speed` keep their solo round.

## How it works

- `yamlengine.FollowTemplate(base, template, existing, haveBase)` is the merge.
  A unit is a mapping key, descended into while the file, template and base each hold a mapping or nothing there, and otherwise compared whole, so a scalar or a sequence is one unit.
  With a base, a unit the file holds as the base does (absence included) takes the template's value, whether that gains, changes or removes it.
  Any other unit is the operator's and is kept, listed under `Differs` when the template holds a third value.
  Without a base, a unit the file lacks is added, a differing value is kept and listed under `Differs`, and a unit only the file holds is kept silently.
  An unchanged file comes back byte for byte; a changed one is re-encoded, keeping comments and node styles but dropping blank lines between entries.
- The base is the template as the reconcile last wrote it, kept in `<module>.template-base` beside the module's file in the board's config dir.
  An apply writes it when absent or stale, and the board commit carries it with the file.
  The name has no `.yaml` suffix, so no module reader takes it for a module file.
- `configsync.ReconcileHubWideAt` runs the merge for each seed-only module over a present board file, after `Migrate`.
  `Result` gains `Updated`, `Differs`, `WroteTemplateBase` and `ChangedFilesRel()`; a dry run, including `lym config reconcile --layer default`, reports `updated` and `differs` too.
- `hubreconcile` mails the hub's orch one line naming each unit added, updated, removed or kept against the template, so the operator learns of a template change.
  The reconcile that a binary change, a start verb or a clone runs applies the merge, so no hub needs a hand step.
- A worktree's copy of a seed-only file is never rewritten.

The seed-only modules are `models` and `profiles`; both follow.

## Decisions to review

- A hub file with no base yet (every hub today) cannot tell which template it was seeded from, so the merge uses the no-base rules above rather than guessing a base.
  On the loomyard hub this adds `code_reviewer_fan: ""` to the `docs` and `speed` blocks and leaves `light` as it is, with the fan.
  The first apply then records the current template as the base, so a stale value the hub still holds from an older template is kept and reported under `differs`, never updated.
- `configreg.Module.Lacking` and `runprofile.LackingTemplateBlocks` are replaced by `Unsettled` and `runprofile.UnsettledFormerBlocks`.
  Adding a template's missing blocks is now the merge's job, so the hook keeps only the former-block-beside-a-differing-current-block pairs, which only the operator can settle.
- A template base that does not parse fails the reconcile naming the base file, with the way forward to delete it; the next reconcile then treats the module's file as having no base.
  A corrupt base is not read when the file has none to follow.
- A failure naming a template base carries no module, since `lym config <module>` does not repair it.
- The docs describe the new rule where they said a present seed-only file is never filled or pruned: `internal/runprofile/doc.go`, `internal/hubreconcile/doc.go`, `internal/configsync`, `internal/configengine`, `internal/configcli`, `docs/shared-libs/yamlengine.md`, `docs/overview.md`, `PATTERN.md` and `pattern/PATTERN-config-layers.md`.

## To check by hand

Not run: a reconcile against the real loomyard and Models hubs.

- `lym config reconcile --layer default` on the loomyard hub: the dry run should list `code_reviewer_fan` added under `docs` and `speed` and no change to `light`.
- Apply it and confirm the board commit holds `profiles.template-base` and `models.template-base` beside their files, and that the orch receives the mail line.
- Run it again and confirm it reports nothing.
- On the Models hub, after its orch removes the key from `light` by hand, confirm the reconcile finds hub and template equal there.
