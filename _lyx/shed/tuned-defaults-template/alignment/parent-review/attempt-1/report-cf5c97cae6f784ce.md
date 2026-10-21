---
verdict: APPROVED
---

## Scope against the board entry

The design does what `tuned-defaults-template` asks, no more and no less.

- The split's template values are all in D1: every `loom` value the entry lists, `shed.driver_kind: go`, `darn`'s reviewer and fixer models and fans, `orch`'s token thresholds, `webster.kickstart_pack: false` and `batcher`'s budget.
  The entry's `claude` "timeouts" are `prompt_cache_ttl_roles`, the only value in which loomyard's `claude.yaml` differs from the template, checked against the hub file.
  `models` and `profiles` hold no value that differs from their templates, so the split's `models` line moves nothing.
- Hubs already cloned follow the template (D3's one-time strip against the current template and the former values), and loomyard's and quarry's default layers drop the entries equal to the template through that walk, with no hand edit.
- The kept entries (`board` labels, `darn.verify`, `landing.publish_verify`, `landing.deploy`, `lenses`' added entries, `gate.slots`) stay in loomyard's hub files.
  Adding `board` kinds and `language.skills` to them follows the entry's own test, since both name Go or lyx.
- The entry's "loomyard's default layer holds only the kept entries" is met for every module but the seed-only `models.yaml` and `profiles.yaml`, which the record keeps whole as the operator's registry (Auto-mode assumptions).
  Neither holds a value that differs from the template, so keeping them changes no resolved value, and the entry does not ask for any change in how seed-only files work.
- The record reads "Everything the operator tunes in loomyard is by default the template everywhere" as the end state this migration reaches, and puts later tunings out of scope as template changes in tasks of their own.
  The split, which spells out the work, asks for no mechanism that carries a later hub tuning into the template, so this is no goal point left out.

No other board entry is duplicated or contradicted.
`kickstart-pack-remove` retires the key D1 sets to `false`, which matches what the hub already does until then; the record names the former-values table entry this leaves behind as an open risk.
`lenses-language-sets` stays out of scope, and the record names the fan names a future lens set must keep.
`shed-arbiter-all`'s principle of shed defaults that a module's config overrides fits `driver_kind: go` in the `shed` template.
`machine-handover` is named only as the planned work behind the older-binary risk.

## Overlaps with other runs

No parent commits are listed.

- `origin-reconcile` (row 16, Merge-In).
  It changes `hubreconcile`'s `reconcileHubWide` so the board push becomes the detached board sync, and `hubforge`'s clone helper so it pushes the clone's board commit.
  D3 adds the base strip and the marker to the same function's board commit and failure restore.
  The two change different behaviours of the same function: the commit, restore and marker stay in the walk, and only the push moves out.
  Landing order: `origin-reconcile` lands first (row 16 against this run's row 3), and this run takes its changes at Merge-In.
- `test-tiers` (row 3, Discussion-Write).
  It changes the `gate` template and retires `darn.verify` and `landing.publish_verify`, both of which this run keeps as loomyard's own with no edit; this run's acceptance example names `darn.verify` as a surviving unit, which moves to another kept unit if `test-tiers` lands first.
  Its open risk about seeded `gates` lists overriding the orch's values is resolved by D2's sparse base rather than contradicted, and both records name the other's `configsync` change as a merge risk.
  No D1 key is retired or changed by `test-tiers`.
  Landing order: a tie, both on row 3.
