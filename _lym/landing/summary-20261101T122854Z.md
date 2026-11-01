# Make wide the batcher template's active profile

`internal/batcher/template.yaml` gains a `wide` profile and names it in `active:`, so every hub groups Webster's cards under `wide` unless it chooses another profile.
`wide` is the `cautious` cost profile with `budget: 300000` in place of 200000: the same batchifier, `max_cards` and estimator weights.
`cautious` and `identity` stay in the template as profiles a hub may pick; no estimator weight changes.

The evidence is `docs/research/batch-budget-trial-2026-11-01/README.md`: over 23 `wide` runs, cost per card was level with `cautious` overall, lower on runs of 10 cards or more, and no quality loss showed.
This hub has run `wide` from its own `batcher.yaml` since 2026-10-22.

## Design decisions

- **`wide` is a full profile, listed before `cautious`.**
  Profiles have no inheritance, so `wide` carries every key.
  The per-key comments sit on `wide`, the active profile, and `cautious` carries one comment naming how it differs.
- **A hub file holding the old default moves to `wide`.**
  `internal/configsync`'s former template values now list `active: "cautious"`, so a not-yet-sparse hub or worktree file that holds the old template's `active:` has it stripped as a former default during reconcile and inherits `wide`.
  Without it, such a file would keep `active: "cautious"` as if the operator had chosen it, and the change would not reach every hub.
- **The migration's orientation source stays `cautious`.**
  `MigrateConfig` reads the orientation to add from the template's `cautious` profile; both profiles hold the same value, which a test now pins.

## Docs

The comments and docs naming `cautious` as the template default now name `wide`: the `internal/batcher` package doc, `DefaultName`'s and `Active`'s comments, the `internal/websterengine` package doc and `docs/overview.md`.

## Tests

- `TestActive` asserts that the template resolves `wide`, and its error rows now break the `wide` profile.
  The row that selects `cautious` explicitly still resolves `cautious`, so the profile stays usable.
- `TestProfileWeights` asserts that the template's `wide` and `cautious` weights are equal and carry the measured start coefficients.
- The configsync strip and retired-key rows and the webster CLI batcher-selection test follow the new template.

## Note for the operator

This hub's `_bolt/_lym/config/batcher.yaml` now repeats the template: its `wide` profile and `active: wide` equal the template's.
Removing it is a hub config change, outside this branch.
