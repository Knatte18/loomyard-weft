```yaml
lens: affected-set
outcome: reported
```

Lens: affected-set. For each change the cards make, I listed what reads or asserts it and which cards cover it.

**Change set and covering cards**

- **Card 1 (`SessionUsage` split; usage read at every outcome).**
  - Readers and asserters found: `wait.go:1932` finalize; `doc.go:288`; `claudeengine/usage.go:278` with `usage_test.go:375`; `wait_test.go:1589/1637`; `shedadapters/roundusage.go:143-151`; `bouncer_usage_test.go:179`.
  - Covered by card 1. `bouncer_usage_test` uses a fake shuttle result, so the change does not break it. Held.
- **Card 13 (plain promote of a discussed note is refused; `--recipe` flag; `validateLabels` rule).**
  - Readers and asserters found: `boardcli/cli.go:180-190` (promote help and examples); the `Promote` tests in `store_test.go`, `board_test.go`, `handles_test.go`, `taskguard_test.go`, `runlock_test.go`, `slugrule_test.go`, `draft_test.go` and `labels_test.go:143`; `boardcli/verbs_test.go:57-106`. All of these use undiscussed notes, so they are unaffected.
  - Refusal texts that tell the user to run a plain `lym board promote` on a note: `boardengine/store.go:373`, `store.go:377`, `task.go:300`, `task.go:307`, `adoptcli/adopt.go:601`, and `adoptcli/doc.go:15`.
  - Card 13 covers `boardcli` and the `validateLabels` tests. No card covers those refusal texts (finding F1).
- **Card 14 (`checkNewSlugs` / `newSlugError` change signature).** The one caller is `board.go:152` and the tests are in `slugrule_test.go`. Both are covered by card 14. Held.
- **Card 9 (loom keys `raddle` and `raddle_auto_approve`).**
  - Readers found: `loomengine/config.go`, `config_test.go`, `loomcli/wiring_test.go`, `webstercli/wiring_test.go`, `docs/overview.md`.
  - Card 9 covers `config.go` and its tests. No card lists the loom key list in `docs/overview.md`; card 27 edits only the recipe lists and verbs there (finding F2).
- **Card 7 (new `report` config module).** The reader is `configreg.go:157`'s module list. Covered by card 7. It has the same docs gap as card 9 (folded into F2).
- **Card 23 (recipe `raddle` joins the closed recipe lists).**
  - Lists found: `shedrun/seed.go:88`; `battencli/wire.go:556/562`; `loomrecipe/names.go:29` (`RecipeEngines`), `:63` (`RowNames`), `:77` (`Rows`), `:101` (the `AmendmentTarget` doc comment), `:117` (`embeddedRecipes`); `battencli/cleanup.go:154`, a branch for motif only.
  - Card 23 covers every one except the `AmendmentTarget` doc comment (finding F3). The `cleanup.go` branch removes a done motif run before a beam is created over the same slug; a raddle run never becomes a beam under its own slug, so it is not in the set.
- **Card 23 (`finishesAsSketch` becomes false for a recipe with a `RaddleFile` row).**
  - Readers found: `loomcli/wiring.go:211-219`, `recipekeyed_test.go`, `loomcli/doc.go`, `boardengine/doc.go`, `boardengine/sketch_test.go`.
  - Card 23 edits `finishesAsSketch` and `TestRecipeKeyedWiring` but lists `internal/loomcli/doc.go` only under Uses, so that file's account of a run with no Finalize row ending as a Sketch goes unedited (finding F4).

**Findings**

- **F1.** LOW · consistency · card 13 (`13-board-promote-raddle.md`).
  - Card 13 makes a plain `lym board promote` of a discussed note with a stored record a refusal.
  - Five refusal sites still offer that plain promote as the way forward for any note: `internal/boardengine/store.go:373`, `store.go:377`, `internal/boardengine/task.go:300`, `task.go:307` and `internal/adoptcli/adopt.go:601`, with the same advice in `adoptcli/doc.go:15`.
  - For a Sketch, that way forward now lands on card 13's own refusal. No card lists these sites.
- **F2.** LOW · consistency · cards 9 and 7 (`09-raddle-paths-and-keys.md`, `07-report-config.md`).
  - `docs/overview.md` lists loom's config keys: a search for `discussion_interactive` matches it.
  - Neither card 9 nor card 27 adds `raddle` or `raddle_auto_approve` there, and card 7 lists no doc for the new `report` module or its `cost_min_runs` key.
- **F3.** NIT · consistency · card 23 (`23-raddle-recipe.md`).
  - The `AmendmentTarget` doc comment at `internal/loomrecipe/names.go:101` enumerates the recipe names Rows accepts, the same list card 23 updates in the `RowNames`, `Rows` and `RecipeEngines` comments.
  - Card 23 misses this one.
- **F4.** LOW · consistency · card 23 (`23-raddle-recipe.md`).
  - Card 23 changes `finishesAsSketch` (`internal/loomcli/wiring.go:207-212`) so a recipe with a `RaddleFile` row does not finish as a Sketch.
  - `internal/loomcli/doc.go`, which the search for `finishesAsSketch` / "makes its entry a Sketch" matches, appears only under the card's Uses. The doc's account of a run with no Finalize row is left stale.

**Tool calls**

1. `grep` for `SessionUsage` / `.Usage` readers and for the callers of `applyDraftRule`, `checkNewSlugs`, `newSlugError`, `validateLabels` and `Promote`: the manifest held their definitions but not their readers or tests.
2. `grep` of `roundusage` tests, tests promoting a Sketch, and docs and refusals naming `board promote`: needed for the reader set of card 13's promote change.
3. `grep` of `verbs_test` promotes, the round-usage test assertions and the `RecipeMotif` lists: needed for card 23's closed lists and card 1's asserters.
4. `sed` of `loomrecipe/names.go`, `battencli/cleanup.go` and `bouncer_usage_test.go`: to judge whether each list or branch belongs in the set.
5. `grep` of the loom-key docs, the config module lists and the docs describing the Sketch finish: the reader sets for cards 9, 7 and 23.
