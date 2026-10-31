```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the set searches behind F3, F5, F6, F7 and F12 to check that the fix covers every member.

Findings I left unchecked: F1, F2 and F10. Their sets are small and the context already lists them (`darnVerifyGatedRows`' rows, the two description-commit closures, `reportengine/doc.go`'s darn line), so I spent no tool call on them. F4, F8, F11 and F13 have no set premise.

**MEDIUM — card 6, the new "Readers of the moved keys outside the module follow" bullet and its Edit list.** The F5 set has more members than the three the review named, and these tests break once the keys leave `loom.yaml`'s template or `implementerEntry`'s texts change:
- `internal/shedrecipe/entries_implementer_test.go:83,85` asserts the way-forward text `such as "loom.implementer"`. Card 6 now rewrites that text to `tapestry.implementer`, so this test is the direct twin of the fix and is not a target. Lines 19, 25 and 55-91 also name `loom.implementer`.
- `internal/configreg/settinglookup_test.go:58,80-83` calls `lookupOnce("loom", "plan_auto_accept")` and writes `plan_auto_accept` into the local `loom.yaml`.
- `internal/testkit/envkit/envkit_test.go:56-61` calls `TemplateSetting("loom", "plan_auto_accept")`, which loom's template no longer answers.
- `internal/configreg/configreg_test.go:196-197` has rows `{"loom", "plan_lenses: twill-plan\n", ...}` and `{"loom", "plan_reviewer_fan: ...", ...}` against module loom.
- `internal/battencli/runsettings_test.go:52` wants `wantDocs: {"loom": "code_max_bounces: 1\nplan_max_bounces: 3\n"}`. Card 6's `KeyMoves` routes `plan_max_bounces` into the `tapestry` document instead.

**LOW — card 6, same bullet.** These comments, help texts and fakes keep naming the old module and would read stale, though they still pass:
- `internal/shedrecipe/entries_planwrite.go:74` says "loom.yaml's plan_lenses".
- `internal/configcli/doc.go:57` gives the example `lym config loom --set 'plan_reviewer_model=...`.
- `internal/boardcli/cli.go:393` has the help example `"tapestry":{"loom":{"plan_max_bounces":1}}`.
- `internal/shedbuild/entrygate_test.go:20,54` and `build_test.go:95` use `loom.plan_auto_accept` with `lym config loom ...` text.
- `internal/loomrecipe/reviewbudget_test.go:222-223` sets Plan-Review's `ConfigFile: "loom.yaml"`.

**LOW — card 9, the added doc-sweep sentence and its Edit list.** The F6 sweep still misses two files:
- `internal/loomrecipe/doc.go:27` says `TapestryRetiredRows` maps `Loom-Preflight` onto `Tapestry-Preflight`. Card 9 changes that mapping to `Preflight` and does not list the file. Card 8's package glyph runs earlier, and card 10's edit of the file is unrelated.
- `internal/loomshed/gatefindings_test.go:6`'s file comment names Tapestry-Preflight. Card 9 deletes only `TestTapestryPreflight_StuckSurfacesItsFailures` from that file.

The other sets are covered:
- **F3:** the `DiscussionHistory` readers are in `TestWireLightweight_FillsThePathsWithoutLoadingAnyConfig` and `TestWire_DarnRecipe`, both card 13 targets.
- **F7:** `citations_integration_test.go` is under the card 13 target `TestValidateLoomDiscussion_Citations`. `docs/overview.md:503` is under card 13's overview edit.
- **F12:** the remaining `RecipeOverride` callers sit in functions card 5 already targets. The rest of the F6 list is covered by card 9's existing targets.

Tool calls:
1. Grepped the repo for the thirteen moved tapestry keys. My context held only the three readers the review named.
2. Grepped for Tapestry-Preflight and CheckSeed. My context held only the sites the review named.
3. Grepped for the deleted validators, the `DiscussionHistory` readers and the recipe-override wording (F7, F3, F12). My context held only the reviewed lines.
4. Read the F5 members outside card 6's targets, to see which name the keys under loom.
5. Read the F6 and F7 members outside card 9's and card 13's targets.
6. Looked up which test holds the F3 lines and read the F12 callers outside card 5's targets.
