---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/plan/02-buttons-config-keys.md
    summary: >-
      The reed template ships task and beam as lists of mappings, so configengine rejects any hub-base or worktree override of those groups that does not repeat the template entries position by position; the card's mapping-group load row fails and D2's acceptance cannot hold
    fix: >-
      Edit card 2 so reed declares the three button groups as open maps, mirroring how darn declares tree_scope.
      Targets: add under Create `plan:internal/reedengine#ConfigOpenMaps` -> `func ConfigOpenMaps() []string`; add under Edit `internal/configreg#Modules` and `internal/configreg#TestLookup`; add under Uses `internal/configreg/doc.go` (the doc-go-unlisted check demands it once configreg code is edited).
      Intent: `ConfigOpenMaps` returns `buttons.task`, `buttons.beam` and `buttons.prime`; `ConfigSpec` sets `OpenMaps: ConfigOpenMaps()`; the reed row of `Modules` gains `OpenMaps: reedengine.ConfigOpenMaps()`, which `TestModules_SeedOnlyAndHubWideFlags` requires to equal the Spec's open maps.
      State why: without open maps `configengine.Load` runs `yamlengine.MissingKeys` over the template's list elements, so a layer's group must repeat `buttons.task[N].label`, `.run` and the confirm key of the same position.
      Replace the sentence "so no load ever fails on a button's content" with one saying a group of any shape loads once the three paths are open maps, while `buttons` itself must stay a mapping.
      Tests: `TestLookup` gains a `reed` row whose `wantOpenMaps` is the three paths; `TestLoadConfig_TemplateDefaultsResolve` gains a row where a worktree file `buttons: {task: [{label: x, run: y}]}` loads and the resolved `task` group is that one entry, replacing the default group whole.
      Keep the existing rows for `buttons: null` (still ErrInvalid), the mapping group `buttons: {task: {label: x}}` (loads) and the local-layer refusal naming `buttons.task`; a probe confirms all three behave so under the open maps.
      Checked with `lym loom validate-plan --overlay`: the card 2 text with these targets adds no blocking finding over the baseline.
  - id: F2
    severity: LOW
    class: consistency
    location: _lym/plan/02-buttons-config-keys.md
    summary: >-
      Card 2 lists docs/overview.md under Extends although it appends words to an existing tree line, and card 9 does the same for the existing tmux feature sentence of internal/toolfloor/doc.go
    fix: >-
      In card 2 move `docs/overview.md` from Extends to Edit, since the card changes the existing `internal/desktophook/` tree line.
      In card 9 move `internal/toolfloor/doc.go` from Extends to Edit, since the card adds items to the existing sentence that lists the tmux features found.
      Both moves were checked with `lym loom validate-plan --overlay` together with F1's targets and add no blocking finding.
  - id: F3
    severity: NIT
    class: consistency
    location: internal/reedengine/bar_integration_test.go:3
    summary: >-
      The file header of bar_integration_test.go still says two-line status bar after card 5 makes the bar three lines, and no card names it
    fix: >-
      Add one sentence to card 5's Intent: the header comment of `internal/reedengine/bar_integration_test.go`, which says it expands the two-line status bar, reads three-line.
      Do not add the file as a target: its test functions are already listed as members, so `redundant-file-target` refuses the file glyph beside them.
---

### [BLOCKING] Open maps missing: the list-of-mappings template defeats the config loader

**Location:** `_lym/plan/02-buttons-config-keys.md`, the reed bullets of the Intent and the `TestLoadConfig_TemplateDefaultsResolve` rows.

**Issue:** Card 2 ships D8's default set in `template_posix.yaml` and `template_windows.yaml` as `task` and `beam` lists of mappings, declares no open map, and says the carried `yaml.Node` means "no load ever fails on a button's content".
`configengine.Load` does not stop at the `yaml.Node`: after `FillMissing` it runs `yamlengine.MissingKeys(template, filled, OpenMaps...)`, and the template's list elements contribute leaf keys such as `buttons.task[0].label`.
`internal/configengine/doc.go` and `yamlengine.md` document that a key missing inside a list element returns an error marked `ErrInvalid`.
No existing reed template holds a list of mappings, so nothing in the repo showed this yet.

I drove the real loader with a scratch program over a reed-shaped Spec (template `buttons.task` of two entries, the second carrying `confirm`; `beam` of one entry; `prime: []`), in hub mode with a sparse base:

- worktree file `buttons: {task: {label: x}}` fails with `missing keys: buttons.task[0].label, buttons.task[0].run, buttons.task[1].confirm, buttons.task[1].label, buttons.task[1].run`; the card's own test row says this loads with no error.
- `buttons: {task: [{label: x, run: y}]}` fails with `missing keys: buttons.task[1].confirm, ...`; `task: null`, `task: []`, `task: [x]` and `beam: {}` fail alike; the hub base file fails the same way.
- Only `prime` takes any shape, because the template's `prime: []` has no elements.
- With `OpenMaps: [buttons.task buttons.beam buttons.prime]` every one of those loads, the upper layer replaces the group whole, the template fills the groups a layer omits, `buttons: null`, `[1]` and `x` still fail on shape, and the local layer still refuses `buttons: {task: []}` naming `buttons.task`.

Consequences against the answer key:

- D2's acceptance (a group written as a mapping in `reed.yaml` loads and is skipped at render, `reedengine.LoadConfig` loads, `lym reed up` succeeds) cannot be met; the card's test row for it fails when its implementer runs it, and the card carries no halt rule or alternative.
- An entry that lacks `run` or `label` as a key, or a hub that tunes `task` or `beam` to a shorter list, makes every `reedengine.LoadConfig` caller fail, which is the cost D2 says a bad button must not have.
- D2's "the hub's base `reed.yaml` holds only the groups it tunes" and "adding a button is one config entry" hold only if a tuned group may differ in length and keys from the default.

The desktop side is unaffected: its template groups are empty lists, so no list element contributes a key, which the probe confirmed.
The registry also holds the open maps a second time (`Modules`' entry, compared with the Spec by `TestModules_SeedOnlyAndHubWideFlags`), so the fix must touch `internal/configreg` as `fix` says.

### [LOW] Extends on lines the cards rewrite

**Location:** `_lym/plan/02-buttons-config-keys.md` (Extends, `docs/overview.md`) and `_lym/plan/09-button-click-binding.md` (Extends, `internal/toolfloor/doc.go`).

**Issue:** The rubric reserves `Extends` for adding to a ref without changing anything already there.
Card 2 says the `internal/desktophook/` tree line "gains" a clause, which changes the existing line at `docs/overview.md:274`.
Card 9 says the list of tmux features found "gains" two items, which changes the existing sentence in `internal/toolfloor/doc.go`.
The other `Extends` entries (new paragraphs or sentences beside existing ones) are correct.

### [NIT] Stale two-line header in the bar integration test

**Location:** `internal/reedengine/bar_integration_test.go:3`.

**Issue:** Card 5 edits `TestBar_LineTwoListsSessionsPrimeFirst` and `TestBar_LineTwoShowsTheCheckMarkBeforeTheClock` in this file and changes every other "two-line bar" sentence it finds (`bar.go`, `windowsize.go`, `doc.go`, `windowsize_test.go`), but the file's own header comment, "expands the two-line status bar against a real tmux server", stays false.

## Rejected

- Focus 1, working directory of the click line: not a defect.
  Probed on tmux 3.6: `run-shell -c '#{@g}'` starts in the home directory, so card 9's `cd #{@lym_button_dir}` inside the line is needed and right; a run directory `a#b c` and one holding `%` reach the shell intact through `#{E:@option}` because tmux substitutes the option value without expanding it again; `##` in the option or in a `run-shell` command expands to one `#`, and `%` in a `run-shell` command is not strftime-expanded.
  D7's literal text names `run-shell -b -c`, which would start the verb in the wrong directory; the plan carries D7's purpose, and I read this as the plan being right.
- Focus 1, working directory of the confirmed line: not a defect.
  A `confirm-before` over `run-shell -b "<line>"` with the directory `a#b c`, `#` doubled as `tmuxQuote` does, ran `pwd` in `.../a#b c` against a real attached client.
- Focus 2, worktree-parent rows: not a defect.
  `ParentOf` (read in `internal/hubgeom/parentof.go`) returns the prime when `ParentBranch` equals the main worktree's branch or `ParentWorktree` equals its directory name; card 4's live-pair and `gone-beam` rows set `parent_worktree` and a `parent_branch` off the prime's branch, and say why.
  The tier0 rows read only the place from the origin record and never call `Beam`; no tier2 row sets a parent field.
- Focus 3, tests reading the status option or line count: covered.
  The readers are `TestPinGeometryOptionsLocked` and its failure-count subtests (8, 9, 8 today), `TestAttachArgv_PreflightOnAKnownGoodSession` (12 + shell pins today; the 13, 14, 15 and 16 across cards 5, 6, 7 and 9 add up), `TestBindingsBuildStamp`, `bindingsFixture.click` with the follow, odd-session and `status 2` steps of `TestBindings_StatusBarClicksSelectAndZoomByRangeType`, the two line-2 bar tests, `TestStatusFormats_SessionLoopsTakeOneFormat` and the reedcli smoke read of `#{status}`.
  The other tests that expand a status format read `status-format[0]` or `[1]` only; no production code maps `#{status}` to reserved rows (the reedcli test's Windows branch inlines a generic mapping).
  The one stale item is F3.
- Focus 4, strand handling: not a defect.
  Only an entry whose spec `strand` is true reads or requires `@lym_strand_name` or a carried name (card 8 step 3); a `window: agents` entry without `{strand}` runs without one, the confirmed line passes `--strand` only when non-empty, and a non-strand confirmed call ignores a stray `--strand`.
  Card 8's Intent states the bound of `--confirmed`, `--strand` and a direct unconfirmed call; `--hash` is covered as part of `--confirmed`, and `--socket` and `--tmux` add nothing a caller that can run the verb could not do by running a program, so I raise no finding for their silence.
- Card 6's `TestUp_BootValidation` row: kept.
  It is the only row that drives `Up` against a mapping group, the guard against a future boot-time refusal that D2's acceptance names.
- Card 8 spanning `reedengine` and `reedcli`: kept as one card.
  `RunButton` and `buttonCmd` are one verb, like `FollowSession` and its command, and the card is reviewable as a unit.
