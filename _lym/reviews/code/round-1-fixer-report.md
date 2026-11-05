```yaml
round: 1
fixed: [F1, F2, F3, F4, F5, F6, F8, F9]
fixed_differently: [F1]
disputed: [F7]
deferred: []
```

## Changes

Commits, oldest first. A commit holds each package's edits, so a finding's hunks can sit in two commits: `ErrInvalidBase` (F1) is in the yamlengine commit because the configsync commit needs it.

- `8f620fce` yamlengine: F5, F7, F1
- `5c42ea93` configsync: F1, F6, F8
- `38ffe629` hubreconcile: F4, F6
- `06da434e` configcli and fabriccli tests: F2, F3, F9
- `3eeac077` self-check fixes, below

### F1 — corrupt template base, fixed differently

The review proposed parsing the base in configsync before the call.
I did not make that change: a second parse would repeat `parseMappingRoot`'s rules, and a base such as `null` that one accepts and the other refuses would leave the wedge in place.
`yamlengine.FollowTemplate` now wraps its base failure in the new `yamlengine.ErrInvalidBase`, and `ReconcileHubWideAt` turns it into a `FileError` whose `Path` is the template base and whose message says to delete it.
The `TestReconcileHubWideAt_SeedOnlyFileFollowsItsTemplate` table has a corrupt-base row asserting the `FileError` path, the "delete it" text and an untouched hub file.

### F2

- `--layer default --apply`: the test commits a stale base, applies, and expects `profiles.template-base` in the Bolt commit beside the files.
- `lym fabric reconcile`: the test drops the base from the board, reconciles, and asserts it is tracked again.
- Clone: the test asserts the base is tracked and recorded as a `file_written` mutation.

### F3

The profiles scenario of `TestConfigOverRealHub` plants a base older than the template, an operator-tuned value and a former-name pair, and asserts `updated`, `differs` and `unsettled` beside `removed` in the dry-run envelope.

### F4

`TestEnsure_CarriesATemplateChangeIntoASeedOnlyConfigFile` asserts the removed, updated and kept-the-hub's mail lines.
It then walks a second time and asserts nothing more is mailed.

### F5

Two `TestFollowTemplate` rows: a sequence at the base's value takes the template's, and an edited sequence is kept whole and listed under `Differs`.

### F6

- `TestReconcileHubWideAt_SeedOnlyFileFollowsItsTemplate` has a row whose template base is a directory, asserting a `FileError` naming the base path.
- The walk-failure row of `TestEnsure_FailedWalkNamesTheFileAndLeavesTheTreeAsItWas` drops the profiles base first, so the walk writes it before the injected failure and the restore must remove it.
  The row covers removal of a written base, which the finding allowed; restore of prior base bytes has no row.

### F8

The `TestReconcileHubWideAt_MigratesLegacyFabricConfig` comment is back above its function.

### F9

`configcli/hubwide.go` and `configcli/doc.go` scope "never fills" to a base that is not seed-only.

## Disputed

- **F7, ground `premise false`.**
  The review says yaml.v3 turns the encoder's default width into 80 and folds long flow mappings.
  `gopkg.in/yaml.v3@v3.0.1/apic.go:111` initialises the emitter with `best_width: -1`, nothing in the package calls `yaml_emitter_set_width`, and `emitterc.go:334` turns a negative width into `1<<31 - 1`, so the default never wraps.
  A `TestFollowTemplate` row with a flow mapping past 80 columns comes back on one line with no code change, and the tier-0 gate passed it.
  Two parts of the finding stood: I kept that row as a regression guard, and the row showed blank lines between entries are dropped, so the `FollowTemplate` doc comment now says so.

## Deferred

None.

## Self-check

Four forks ran. Findings that name the same text are merged.

- fix-code-correctness, LOW: a corrupt base's `FileError` reached `WorktreeError.Error()`, which suggested `lym config profiles --layer default`; that command reads the same base and fails the same way.
  Merged with fix-proposal's LOW on the same text. Fixed with my own change: `worktreeErrorFor` leaves the module empty for a template base file, so the message says to fix the file and re-run.
  `configengine.IsTemplateBaseFile` is new, and a row in `TestEnsure_FailedWalkNamesTheFileAndLeavesTheTreeAsItWas` asserts the message holds "delete it" and no `lym config`.
- fix-consistency, LOW: `docs/overview.md:378` said "never fills a present one" unscoped.
  Merged with fix-proposal's LOW and fix-affected-set's LOW on the same line. Fixed: scoped to a present one that is not seed-only.
- fix-affected-set, MEDIUM: `internal/hubreconcile/doc.go` said every write to a base file never fills a template value, which contradicts a seed-only file following its template.
  Fixed: the base-file definition excepts a seed-only board file, and a sentence says what the walk does with it.
- fix-consistency, LOW: `docs/shared-libs/yamlengine.md` had no entry for `FollowTemplate`.
  The gap predates this round and the rubric excludes it, but a short entry was cheaper than arguing it. Fixed with a new entry that includes `ErrInvalidBase` and the dropped blank lines.
- fix-affected-set, LOW (the `configsync.go` `ReconcileAll` doc, which it judged correct for a per-worktree file): not a finding; no change.
- fix-proposal, note on F6: the restore row covers removal of a written base, not restore of prior bytes.
  The review allowed "restored or removed"; no change.

No fork returned without a report. The lens reports are in `round-1-fixer-report.lens-<lens>.md` beside this file.

## Verification

`lym gate test` over `yamlengine`, `configsync`, `configengine`, `hubreconcile`, `configcli` and `fabriccli`, tiers 0 and 1: no failures.
The new corrupt-base row and the base-restore row of the walk-failure test ran and passed in tier 1.
