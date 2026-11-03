```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness. I walked each function and branch the diff adds or changes and ran one concrete input through it.**

**List (function or branch: case run, verdict)**

- `MigrateConfig`, empty input: `yaml.Unmarshal` leaves a zero node, so the `Kind != DocumentNode` guard returns `existing, nil, nil`. Correct.
- `MigrateConfig`, header-only or comment-only document: the same guard returns it unchanged. Correct.
- `MigrateConfig`, `defaults: {loom: solid}` plus `solid: {loom: {loom: {...}}}`: the key node is renamed at depth one under each top-level entry only. The inner module key `loom` is untouched. Operator values are kept. Correct.
- `MigrateConfig`, former block beside a current one: the former block is deleted with `slices.Delete(at, at+2)` and the current block is kept. The next rename re-looks-up its index, so no stale index. Correct as written (see F2).
- `MigrateConfig`, scalar-valued entry (`light: 3`): skipped by the `recipes.Kind` check. Correct.
- `MigrateConfig`, unparseable input or non-mapping root: returns an error marked `ErrInvalid`. The caller reports a `FileError` before any write. Correct.
- `MigrateConfig`, no former name: returns the original bytes with no lines, so no write. Correct.
- `MigrateConfig`, renamed document: re-encoded through `yaml.Encoder` at indent 2. Values and comments are carried, but the file is re-serialized (see F2).
- `LackingTemplateBlocks`, empty or header-only file: `hub` is a nil map and nothing is listed. Correct.
- `LackingTemplateBlocks`, entry absent from the hub or not a map: skipped. Correct.
- `LackingTemplateBlocks`, `solid` seeded before `sampler`: `solid.sampler` is listed, and the result is sorted. Correct.
- `LackingTemplateBlocks`, unparseable input: marked `ErrInvalid`. Correct.
- `ReconcileHubWideAt`, absent seed-only file with no prime copy: the template is used verbatim, `changed` stays `true` from `fileAbsent`, and `break` leaves the switch. Correct.
- `ReconcileHubWideAt`, absent seed-only file seeded from a prime or legacy copy: it runs Migrate and Lacking, and `changed` is `true`. Correct.
- `ReconcileHubWideAt`, present `profiles` file:
  - Migrated lines set `changed`, and the file is written once with `Applied` true.
  - With no lines, nothing is written and `Lacking` is still returned. Correct.
  - A dry run writes nothing. Correct.
- `ReconcileHubWideAt`, present per-worktree `models` file (the removed early return): it takes the SeedOnly branch with no hooks, so `changed` is false, `Seed` is `SeedHub` and the result is unchanged. Correct.
- `ReconcileHubWideAt`, absent per-worktree default without `SeedDefaults`: the early return for `!m.HubWide &amp;&amp; fileAbsent &amp;&amp; !SeedDefaults` is kept. Correct.
- `ReconcileHubWideAt`, Migrate or Lacking error: `reconcileErr` becomes a `FileError` and returns before the write, so the unparseable file stays untouched and the error is clear. Correct.
- `Profiles.Settings`, profile holding only `loom`: the error names the former block and `lym config reconcile --layer default --apply`. Correct.
- `Profiles.Settings`, profile holding neither: the old message. Correct.
- `Profiles.Default`, `defaults` holding only `loom`: returns an error that includes the way forward. Correct.
- `Profiles.Default`, `defaults` holding both `tapestry` and `loom`: the current name wins and there is no error. Correct.
- `Profiles.Default`, a recipe with no default under any name: returns `""` and nil. Correct.
- `prepareRunSettings`, `Default` error: wrapped in `ErrInvalidRunSettings`, so Seed-Child ends Stuck. Correct.
- `reconcileEnvelope` and the `hubreconcile` walk, `Lacking` reporting: the envelope sets `lacking` for every result that carries it, and the walk logs one Warn per module before the `Applied` filter. Both fire when nothing was written. Correct.
- `recipeRenames`, `heldFormerName`: only a recipe that is some rename's `to` can match, so `darn`, `pawl` and `sampler` never match. Correct.

**Findings**

- **F1, LOW, design:** `internal/configsync/configsync.go:980-986`. A present but malformed `profiles.yaml` now makes `ReconcileHubWideAt` return an error. Before this change a present seed-only file was never read. The hub reconcile in a start verb runs inside one `PullThenCommitWritten` and fails as a whole. It restores the prior config, but a hand-broken `profiles.yaml` now blocks every module's reconcile and the start verb. This matches how the non-seed-only modules already treat a malformed file, and the error is clear, so it is a design note only. The new behavior is unpinned in the walk (no test).
- **F2, LOW, design:** `internal/runprofile/migrate.go:80-93`. Re-encoding through `yaml.Node` and `Encoder` keeps comments and values. It does not keep blank lines, flow-style spacing (the test only checks `{...}` substrings) or the original layout. An operator-owned file is reformatted wholesale on the first migrating reconcile, and the drop of a former block beside a current one discards that block's values. The code keeps the current block, which is the one a run reads, and reports the drop as a Migrated line. I did not run the encoder on the shipped template, so blank-line and header-comment preservation is unverified. Not a data-loss defect for the active values.

No BLOCKING or MEDIUM defects found in this lens.

**Tool calls (4)**
1. `configsync.go:505-604`: whether `reconcileBaseWorktreeFile` and `reconcileSparseFile` run Migrate on a seed-only file, which my context lacked. They do not (both gate on `SeedOnly`).
2. `configsync.go:725-765`: `sparseSeed` calls Migrate, but only on the non-seed-only paths.
3. `configreg/runsettings.go:55-104`: whether the new `profiles` Migrate is reachable through run-settings layering. It is reachable only for a `profiles` key that `checkLocalDocument` then refuses.
4. `hubreconcile/walk.go:185-244`: the walk's error and restore path, and where the Lacking Warn sits relative to the `Applied` filter.
5. `configcli.go:585-644`: whether `lacking` is emitted for results that are not applied. It is.

(That is five reads; the first two were issued in one parallel batch with the third and fourth.)
