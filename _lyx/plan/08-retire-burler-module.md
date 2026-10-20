# Card 8 — retire the burler module

**Delete:**
- `internal/burlerengine/config.go`
- `internal/burlerengine/fankeys.go`
- `internal/burlerengine/template.yaml`
- `internal/burlerengine/config_test.go`

**Edit:**
- `internal/configreg#Module`
- `internal/configreg#Modules`
- `internal/configreg#KeyMove`
- `internal/configreg#KeyMoves`
- `internal/configsync#Result`
- `internal/configsync#moveKeys`
- `internal/configsync#moveFile.takeKey`
- `internal/configsync#ReconcileHubWideAt`
- `internal/configreg#PrepareRunSettings`
- `internal/configreg#TestModules_SeedOnlyAndHubWideFlags`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/configreg#TestLookup`
- `internal/configreg#TestPrepareRunSettings`
- `internal/configsync#TestReconcile_MovesKeysAcrossModules`
- `internal/hubreconcile_test#runStaleBuildReconcile`
- `cmd/lyx#configStrictnessDegradingSet`

**Uses:**
- `internal/lenses#ConfigTemplate`
- `internal/gitkit#Git`
- `internal/gitkit#CommitFile`
- `internal/gitkit#LsFiles`

**Intent:** Retire the `burler` config module and migrate its file into `lenses.yaml` through two declared key moves (D1, D4, D6).

- Delete burlerengine's `config.go`, `fankeys.go`, `template.yaml` and `config_test.go`; after cards 5 to 7 nothing reads them, and `internal/lenses` holds their content and tests.
  `Modules` drops the `burler` entry and configreg its `burlerengine` import, so `lyx config burler` gives way to `lyx config lenses`; `Module`'s `SeedOnly` comment drops `burler.yaml` from its examples.
- `KeyMove` gains `KeepTarget bool`: when set, a target file that holds the key and was not started from the template in the same pass keeps its value whatever it is, the template default included, at a base layer as at a sparse one, and the kept-target line names the dropped and kept keys without rendering their mappings, as `<source> dropped, <target> kept`.
  `moveFile.takeKey` implements it before the template-default comparison; `moveKeys`'s and `Result.KeptTarget`'s comments describe it.
  Bound: only the two moves below set it, so every other move keeps today's template-default overwrite; a target started from the template in the same pass, or one lacking the key, still takes the source's mapping whole, and a move never merges per name.
- `KeyMoves` gains `{FromModule: "burler", FromKey: "lenses", ToModule: "lenses", ToKey: "lenses", KeepTarget: true}` and the same for `fans`, after the existing moves.
  `moveKeys`'s source side does no registry lookup, so a `burler.yaml` is still read and, emptied by the two moves, deleted, at every layer a reconcile walks; the following module loop then reconciles `lenses.yaml` per name, adding every template lens and fan the moved mappings lack.
- Run settings holding a `burler` object, empty or not, are refused as naming an unregistered module: `moveRunSettings` moves its keys into a `lenses` object and leaves the `burler` object, which `PrepareRunSettings`'s module lookup refuses before its empty-object skip.
  The next step is to remove the `burler` object from the entry's run_settings, since both its keys are guard keys a run setting cannot hold, so moving them into a `lenses` object is refused too.
  `PrepareRunSettings` names that step in the refusal's text for an unregistered module that a declared key move retired, one a `KeyMoves` entry names as `FromModule`: `module "burler" is not registered: its keys moved to another module; way forward: remove the burler object from run_settings`.
- `hubreconcile.Ensure` needs no change: the source's move result is applied, so `burler.yaml`'s path joins the files the board commit stages, and the commit stages the deletion of a written path that no longer exists.
- `configStrictnessDegradingSet`'s file header comment names `internal/modelspec` alone as the own-loader module.
  The header sits outside the var's span and `edit-test-file` refuses the `_test.go` path, so this sentence is its carrier.
- Bound: the moves carry only `lenses` and `fans`, the only keys `burler.yaml` holds, so a `burler.yaml` holding any other key keeps it and is not deleted, and no reconcile touches it again once `burler` is unregistered.
  A local `burler.local.yaml` moves into `lenses.local.yaml` as any local-layer move does, which the `lenses` load then refuses naming the local file, since both keys are guard keys; the existing claude local-move row pins that mechanic.
  The way forward for a refused `lenses.local.yaml` is to delete that local file.
- `ReconcileHubWideAt` carries D4: on a dry run the module loop reports no seed additions for a module whose absent board file the same pass's key moves would write, since the moves' values, not the seed, fill that file and the additions are known only once it exists.
  The rule holds for every move target, the claude moves' as well as the lenses ones; an apply is unchanged and reports in `Added` the names the moved values lack.

Tests:
`TestReconcile_MovesKeysAcrossModules` gains an `added` field (each line must appear in some result's `Added`), a `noAdded` field (modules none of whose results carries an `Added` line) and an `unchanged` field (file keys whose content after the run equals the content before), and these rows, each fixture built in the test from `lenses.ConfigTemplate()`:
a board `burler.yaml` of today's hub shape (the template with one lens and one fan dropped, one lens's prose tuned and the seven-lens `twill-discussion`) and no `lenses.yaml`, call `board` with no prime copy, writes a `lenses.yaml` holding the tuned prose, the seven-lens fan and the dropped lens and fan again, leaves no `burler.yaml`, reports both moves and `added` names the dropped lens and fan;
the same files on a dry run write nothing, keep `burler.yaml`, leave `lenses.yaml` absent, report both moves, and `noAdded` names `lenses`;
a `lenses.yaml` equal to the template beside an old-template `burler.yaml` (the template with the six-lens `twill-discussion`) leaves `lenses.yaml` unchanged, deletes `burler.yaml` and reports `burler.fans dropped, lenses.fans kept` and `burler.lenses dropped, lenses.lenses kept`;
a non-default board `lenses.yaml` beside a `burler.yaml` is kept unchanged with the same two kept lines and no `burler.yaml`;
on the worktree layer of a hub (call `hub`) a sparse `lenses.yaml` holding a `fans` mapping beside a `burler.yaml` holding `fans` is kept unchanged, `burler.yaml` is deleted and `burler.fans dropped, lenses.fans kept` is reported;
a board holding a `burler.yaml` and no `lenses.yaml`, seeded from a prime whose worktree-layer `lenses.yaml` holds only a `fans` mapping, writes a `lenses.yaml` keeping the prime's `fans` and burler's `lenses`, deletes `burler.yaml`, and reports `burler.lenses -> lenses.lenses` moved and `burler.fans dropped, lenses.fans kept`.
`TestPrepareRunSettings` gains two rows: a `burler` object holding `lenses`, and an empty `burler` object, each refused as a `*RunSettingsError` for module `burler` naming `not registered` and the way forward `remove the burler object from run_settings`.
`TestLookup` gains a `burler` row not found; `TestModules_SeedOnlyAndHubWideFlags` drops `burler` from the seed-only set and its comment, and `TestModules_EveryTemplateKeyClassified` drops its `burler` entry.
`runStaleBuildReconcile` puts the board in today's shape before the walk, removing the committed `lenses.yaml` and committing a `burler.yaml` holding a tuned `generic` lens and the seven-lens `twill-discussion`, and after the first walk asserts the board tracks `lenses.yaml` holding the tuned prose and a template lens, and no longer tracks `burler.yaml`; its second walk still lands no board commit.

**ImpactSummary:** Removes the burler config module and its files; configreg, configsync's move and the configreg, configsync and hubreconcile tests change, while every lens and fan reader already reads internal/lenses since cards 5 to 7, and moves other than the two new ones apply as today; a board dry run no longer reports seed additions for a module whose absent file a move would write, and an unregistered run-settings module a move retired is refused with a way forward.
