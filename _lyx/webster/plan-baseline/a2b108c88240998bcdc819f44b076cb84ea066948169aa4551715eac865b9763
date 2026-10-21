# Card 6 — sparse-hub-base-reconcile

**Create:**
- `plan:internal/configsync#sparseSeed` -> `func sparseSeed(m configreg.Module, seed []byte, dropFormer bool) ([]byte, error)`
- `plan:internal/configsync#stripBase` -> `func stripBase(m configreg.Module, current []byte) ([]byte, []string, error)`
- `plan:internal/configsync#WriteBaseSparseMarker` -> `func WriteBaseSparseMarker(boardDir string) error`
- `plan:internal/yamlengine#DropTemplateEqualNames` -> `func DropTemplateEqualNames(template, existing []byte, perNameMaps []string) (out []byte, dropped []string, err error)`
- `plan:internal/yamlengine#TestDropTemplateEqualNames` -> `func TestDropTemplateEqualNames(t *testing.T)`
- `plan:internal/configsync#TestReconcileHubWideAt_StripsAnUnmigratedBaseOnce` -> `func TestReconcileHubWideAt_StripsAnUnmigratedBaseOnce(t *testing.T)`

**Edit:**
- `internal/configsync#HubWideOptions`
- `internal/configsync#Result`
- `internal/configsync#ReconcileHubWideAt`
- `internal/configsync#retireHubWideCopy`
- `internal/yamlengine/doc.go`
- `internal/configsync#TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate`
- `internal/configsync#TestReconcileHubWideAt_MigratesLegacyFabricConfig`
- `internal/configsync#TestReconcileAll_HubWideCopy`
- `internal/hubforge#TestNewHub`
- `PATTERN.md`

**Uses:**
- `plan:internal/configsync#formerValuesDoc`
- `internal/configsync#writeModuleFile`
- `internal/configsync#findResult`
- `internal/configsync#lensesFixture`
- `plan:internal/configengine#SparseBaseHeader`
- `plan:internal/configengine#BaseSparseMarker`

**Intent:**
`ReconcileHubWideAt` keeps every base file at the board dir sparse (D2, D4) and carries D3's one-time strip behind a walk-only option.
A seed-only module (`configreg.Module.SeedOnly`) is reconciled exactly as today, its absent file seeded whole and a present one never rewritten.

For every other module:
- A present file runs the module's `Migrate` hook and then `yamlengine.Prune` against the template, which removes keys the template lacks and adds none.
  `Prune` never drops a per-name entry, so for a module with `PerNameMaps`, such as `lenses`, the pruned bytes then pass through `yamlengine.DropTemplateEqualNames`, which removes each per-name entry whose value decodes equal to the template's, with the same equality `yamlengine.Reconcile` uses for its dropped report, and returns the removed paths as `P.<name>`, sorted; they are reported in `Result.Dropped`, so `lenses` keeps dropping a template-equal entry on every reconcile as today.
  `DropTemplateEqualNames` and `mergePerNameMaps` share that one comparison, `sameValue`, and a file it leaves unchanged is returned byte for byte.
  `internal/yamlengine/doc.go` lists `[DropTemplateEqualNames]` beside `[Prune]` in its function list, with its contract, and its Per-name maps section names it among the functions taking a perNameMaps argument.
  `TestDropTemplateEqualNames` in `layers_test.go` is table-driven, with rows for a template-equal name dropped and reported, the same value in another quoting style dropped, a differing name kept, a name only the file holds kept, and a file with nothing to drop returned byte for byte.
  So `Added` stays empty for a present file.
  A file the prune leaves with no unit is written as `configengine.SparseBaseHeader`, never deleted.
- An absent file seeds sparse through `sparseSeed`: from the template it is the header-only base; from the prime's copy it holds that copy's units, after `Migrate`, that differ from the current template and, while the prime worktree has no `sparse` marker of its own (`configengine.SparseMarker(primeBaseDir)` absent, `dropFormer` true), from the module's former value (`formerValuesDoc`); from the legacy files it holds their units that differ from the template.
  An empty result is the header-only base.
  The `Seed` reported is unchanged, and `Added` lists nothing, since no template key is copied in.
  This covers every caller that seeds an absent base: the walk and `lyx config reconcile --apply --layer default` under `SeedDefaults`, and `lyx fabric reconcile` and `lyx fabric clone` for hub-wide modules with open maps or `SeedsFromPrime`.
- `HubWideOptions` gains `StripBase bool`, documented as set only by `hubreconcile`'s walk.
  While `configengine.BaseSparseMarker(boardDir)` is absent, each base file present when the pass began has, after `Migrate`, the key moves and the prune, every unit equal to the template's or to the module's former value removed by `stripBase`, which compares with `yamlengine.Strip` and the module's open maps, so an open map is compared whole.
  A target base the pass's key moves created has its moved units stripped the same way, so a moved unit equal to the template or a former value is dropped on an unmigrated board, as D4 says; the units its sparse prime seed holds are not stripped again, since the seed already dropped what D4 drops and a prime-seeded unit equal to a former value under a marked prime is tuned.
  A base seeded from the prime, the legacy files or the template in the pass is otherwise not stripped, since its seed is already sparse.
  The removed keys are reported in `Result.Stripped`, applied only with `StripBase` and `Apply`, and reported as pending on a dry run or without `StripBase`, so `lyx config reconcile --layer default` shows the strip per module with the keys it would drop and `lyx fabric reconcile`, `lyx config reconcile --apply` and `lyx fabric clone` strip no present base file.
  A base the strip empties is written as the header-only base, never deleted.
  A unit equal to neither the template nor a former value, such as loomyard's `darn.verify` or `board` labels, is kept.
  With the marker present no strip is computed or reported, so a unit later set equal to the template or a former value stays as a pin.
- Under `StripBase` and `Apply`, once every module is written and every registered module's base file exists at the board dir, `WriteBaseSparseMarker` writes the marker, a tracked one-line file saying the board's base files are sparse over the template; an absent base withholds it.
  The marker is written after the files, so an interrupted pass leaves the board unmarked and the next walk strips again.
  `WriteBaseSparseMarker` is exported for `lyx fabric clone`, which writes the marker on a board it seeded from nothing (card 11); a failure is a `*FileError` naming the marker path with an empty `Module`, since the marker belongs to no config module and no `lyx config <module>` edits it; `hubreconcile`'s `WorktreeError` then names only the file (card 8).

`Result.Stripped`'s comment says ReconcileHubWideAt sets it on an unmigrated board, and `Result.Added`'s and `Result.Seed`'s that a hub base seed copies no template key.

`retireHubWideCopy` compares a leftover hub-wide per-worktree copy with the template merged under the hub file, read through `configengine.BelowLayer` over `configengine.Dirs{Board: boardDir}` and the module's spec at the worktree layer, never the raw hub file, so a copy unit equal to a template value the sparse hub file leaves out is not reported `Divergent` and a copy holding only such units is `Retired`; an absent hub file still leaves the copy alone.
`TestReconcileAll_HubWideCopy` gains rows: a copy holding a template default over a hub file lacking that key is retired, and a copy value differing from both the template and the hub file is divergent.

`TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate`'s loom rows expect the seeded or reconciled base to hold only its non-template units (`discussion_timeout_min: 480` from the prime, the header from the template) and no `review_max_bounces:`;
a new row gives a present loom base holding only `stale_key: old` and expects the file present after the apply, holding only `configengine.SparseBaseHeader`; a new row seeds a loom default from an unmarked prime copy holding `discussion_lenses: ""` and `review_max_bounces: 99` and expects only `review_max_bounces: 99`, and a twin with the prime marked keeps both, since `""` differs from the new template; a new row gives a present lenses base holding a template-equal lens beside a tuned one and expects, after the apply, the template-equal lens absent and reported in `Dropped`, the tuned one kept, with no `StripBase` set; the models rows are unchanged.
`TestReconcileHubWideAt_MigratesLegacyFabricConfig`'s row that expected the template-default `pathspec` filled in expects it absent, the seeded file holding only `branch_prefix: hanf/`.
`TestReconcileHubWideAt_StripsAnUnmigratedBaseOnce` is table-driven over a board dir whose base files are written for every registered module, with rows for: a loom base holding the current template whole plus `discussion_lenses: ""` and `darn` holding a tuned `verify`, which under `StripBase` and `Apply` ends header-only for loom, keeps darn's `verify`, reports the stripped keys and writes the marker; the same board without `StripBase` writes no strip and no marker and reports the keys as pending; a dry run with `StripBase` writes nothing; a `claude` base holding the former `prompt_cache_ttl_roles` map whole is stripped to the header, and one holding that map with one role changed keeps it, so the open map is compared whole; a board already holding the marker keeps a unit equal to a former value; a board whose per-worktree `loom` base is absent, reconciled under `StripBase` and `Apply` without `SeedDefaults` so the pass leaves it absent, strips the others but writes no marker; and a seed-only `models` base is left byte for byte.

`TestNewHub`'s `ConfigMaterializedWithoutSeeding` step, which compared the board's seeded `loom.yaml` with the whole template, expects every non-seed-only module's file at the board dir, hub files and repo defaults alike, to be present with the bytes `configengine.SparseBaseHeader`, bar a file the clone's own fixture tunes, and `loomengine.LoadConfig` over the prime to resolve `DiscussionLenses` `twill-discussion`; its comment says a fresh hub's base holds no tuned unit.

`configsync.go`'s header comment and `ReconcileHubWideAt`'s doc drop "the base layer stays full": in a hub every non-seed-only base file is sparse over the template, seeded sparse and pruned but never filled, stripped once by the walk behind the board marker, and only a standalone worktree base and a seed-only file stay full.
`PATTERN.md` gains, under `## Paths and geometry` after `PATTERN-hub-wide-config`, the entry line:

```markdown
- `PATTERN-sparse-hub-base` — Writing a hub's base file (a hub-wide module's hub file or a per-worktree repo default, not seed-only): only the units the hub tuned, never a template fill; only `hubreconcile`'s one-time walk strip drops a unit the template holds, bar `lenses`' template-equal entries.
```

**ImpactSummary:** `ReconcileHubWideAt`'s seed and reconcile of every non-seed-only base file and the hub-wide copy comparison, plus a `yamlengine` per-name drop sharing `Reconcile`'s equality; its four callers keep their calls, the walk opts into the strip in card 8, and the new PATTERN entry binds every later base writer.
