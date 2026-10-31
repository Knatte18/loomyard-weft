```yaml
lens: affected-set
outcome: reported
```

I worked the affected-set lens: for each change the cards make, I listed what reads or asserts it and which cards cover it.

## Per-change affected sets

- **Removed loomrecipe builders and retired-row functions (card 8).** Readers: `loomcli/arm.go` (`armAt`, `recipeRouting`, `newShed`), `darnwiring.go:240` (`wireDarn`), `peers.go:88-90` (`recipeRow`), `wiring_darn_test.go:292-294` (`TestRenameRetiredRows`), `wiring_test.go:430,448` (`TestWire_DefaultConfig`) and `reportengine` `darnPhases`. Doc mentions of `loomrecipe.New` in `cli.go`, `shedcheck/doc.go`, `battenrecipe.go`, `sharedbootstrap.go` and `shedverbs/spec.go` stay valid. Covered by card 8.
- **Merged validators and gates (card 13).** Readers: `gates.go`, `gates_test.go`, `citations_integration_test.go:72-190` (inside `TestValidateLoomDiscussion_Citations`), `parity_test.go:142`, `validate.go`, `entries_gate.go` and `docs/overview.md`. Covered by card 13.
- **`DiscussionHistory` dropped from envkit's `nilLegal` (card 13).** Readers: `envkit.go:54`, the loomcli seam test, and battencli `wire_seams_test.go:37-124` (`intentionallyNil`, checked both ways). Card 13 covers it partly; the battencli entry is missed.
- **Tapestry-Preflight row, its producer and `CheckSeed` removed (card 9).** Readers: the files card 9 lists, plus `shedrecipe/conformance_test.go:36`, `loomengine/seed.go:1,113-121`, `loomrecipe/loomrecipe.go:53-65`, `reportengine/doc.go:130`, `loomcli/bootstrap.go:142`, `loomcli/smoke_test.go:214`, `loomshed/gatefindings_test.go:6` and `pattern/PATTERN-cwd-resolution.md:21`. `orch-template-role.md` is covered by card 16. Card 9 covers the set partly.
- **Plan and implementer keys moved to `tapestry` (card 6).** Readers: `tapestry-recipe.yaml`, `implementer-spec.md`, `docs/overview.md`, `wiring_test.go` and `fansettings_test.go`, all covered. Card 6 misses `configcli/configcli_test.go:1306-1311`, `shedbuild/build_engines_test.go:22,62` and `shedrecipe/entries_implementer.go:30-41` (the example key in its way-forward text). `entrygate_test.go`, `build_test.go:95` and `entries_implementer_test.go` use a fake lookup and are unaffected. `reviewbudget_test.go:222` has an explicit fixture that still passes. Card 6 covers the set partly.
- **Shed driver keys leave `loom.yaml` and `darn.yaml` (card 5).** Readers: `docs/overview.md`, `holdreading_test.go`, `keymove.go`, `runsettings_test.go`, `configsync_test.go`, `darnengine/doc.go`, `tacklerfor_test.go` and `sheddriver/doc.go`, all covered. Card 5 does not list `loomcli/doc.go` or `sheddriver/config_test.go`. Card 5 covers the set partly.
- **Darn Describe table and stencil removed (card 17).** Readers: `describe_test.go`, `darnwiring.go:281`, `stencils.go:26-29,459,569`, `markers_test.go:47` and `landingshed/doc.go:247`. Covered by card 17.
- **Darn board-entry snapshot dropped (card 12).** Readers: `darn-recipe.yaml`, `loom-rubric-code-review.md`, `docs/overview.md`, `darnwiring.go`, `wiring_darn_integration_test.go:68-73`, `wiring_darn_test.go` and `darnrecipe_test.go`. PR-Review's `BoardEntryPath` is optional (`landingshed/deps.go:364-366`, `prsession.go:158`), so it does not break. Pawl's path through `pawlwiring.go:135` stays. Covered by card 12.
- **Darn's drift subject becomes the record (card 15).** Reader: `wiring.go:787` `driftSubject`. Covered by card 15.

## Findings

1. **MEDIUM — design — card 13 (`internal/testkit/envkit#nilLegal`).** Card 13 drops `DiscussionHistory` from `nilLegal` but adds only `Darn.CheckRecord` to battencli's `intentionallyNil`. Batten's `wire` never sets `DiscussionHistory`, so the two-way check in battencli's tier0 `TestWire_EverySeamFilled` fails at card 13's gate. Missed member: `"DiscussionHistory"` in `internal/battencli#intentionallyNil`. Evidence: `internal/testkit/envkit/envkit.go:54`; `internal/battencli/wire_seams_test.go:1,37-124,184-192`; `internal/loomcli/wiring.go:307-310`.

2. **MEDIUM — design — card 9 (removal of Tapestry-Preflight and `CheckSeed`).** Card 9 covers only part of the set that names the row, its engine or `CheckSeed`. It missed:
   - `internal/shedrecipe/conformance_test.go:36`: a registry-keyed `"TapestryPreflight"` row that outlives the deleted registry entry.
   - `internal/loomengine/seed.go`: the file header (line 1) and `VerifySeedOwnership`'s doc (lines 113-121) both describe `CheckSeed` and the Tapestry-Preflight row.
   - `internal/loomrecipe#New`: its doc (`loomrecipe.go:53-65`) justifies the status-path check by `tapestryPreflightEntry`.
   - `internal/reportengine/doc.go:130`: listed only under `Uses`, yet it names Tapestry-Preflight in tapestry's setup phase.
   - Comments at `internal/loomcli/bootstrap.go:142` and `internal/loomcli/smoke_test.go:214`.
   - `pattern/PATTERN-cwd-resolution.md:21`: names `loomengine.CheckSeed` as the third resolution tier.
   
   Evidence: the lines above.

3. **MEDIUM — design — card 6 (moving `plan_*` and `implementer` to `tapestry`).** Card 6 covers only part of the set that reads `loom.plan_*` or `loom.implementer`. It missed:
   - `internal/configcli/configcli_test.go:1306-1311`: sets `plan_reviewer_model` on loom's local layer. Loom's `ValidateLocal` refuses it once the key leaves loom's template.
   - `internal/shedbuild/build_engines_test.go:22,62`: an Implementer row with `implementer_setting: loom.implementer`, resolved through envkit's `TemplateSetting` (`envkit.go:236-239,310`). The lookup no longer finds the key.
   - `internal/shedrecipe/entries_implementer.go:30-41`: its way-forward texts give `loom.implementer` as the example key.
   
   Evidence: the lines above.

4. **LOW — consistency — card 5 (shed driver keys leave loom and darn).** `internal/loomcli/doc.go` and `internal/sheddriver/config_test.go` still name the `loom` or `darn` driver, arbiter or tackler keys, and card 5 lists neither. I did not open `config_test.go`, so whether its assertions break is unconfirmed. Evidence: the grep in tool call 6, which matched both files.

## Tool calls
1. Grep for readers of the removed builders, validators and Tapestry-Preflight/`CheckSeed` names across the repo: the manifest held only the cards' own target lists.
2. Line-level matches of those names in files no card lists: needed to tell real uses from doc mentions.
3. Enclosing functions of four call sites, `seed.go`'s declarations, and readers of darn's Describe and the board-entry snapshot: needed to map hits onto listed members and to check card 12's and card 17's sets.
4. PR-Review's board-entry readers and readers of the moved plan and implementer keys: not in the manifest.
5. Line-level matches of those PR-Review and moved-key readers: to judge whether each breaks.
6. envkit's `TemplateSetting`, the configcli test row, the entry-gate fixture, and readers of the shed driver keys: to confirm the card 6 breakages and card 5's set.
