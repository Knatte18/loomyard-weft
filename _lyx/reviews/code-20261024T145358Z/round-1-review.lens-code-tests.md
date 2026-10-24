```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: per behaviour the diff adds or changes, the test that pins it, then the findings. I made one tool call, a read of the test diffs.

**Behaviour → pinning test**

- Ladder values (rounds, hard cap = rounds + 1, Code 1 in every loom block, darn column): `TestTemplateLadder` in `runprofile_test.go`, one subtest per rung.
- Template names and defaults (`docs`, `light`, `solid`, `speed`, `wide`; `loom: solid`, `darn: light`): `TestLoad` row "no file reads the template's ladder, docs and defaults".
- `docs` is darn-only, with `darn.writer: sonnet[medium]` and one Code round: none. `TestLoad` checks only that the name exists. `TestTemplateLadder` skips `docs`. Nothing asserts the `docs` values or that it has no loom block, so changing `sonnet[medium]` or `code_max_bounces: 1` breaks no test.
- Shape refusals (profile not a mapping, recipe block not a mapping, old module→keys shape, defaults not a mapping): the `TestLoad` rows of the same names.
- A profile named `defaults` is refused: the `TestLoad` row of that name. It passes only because the recipe block's value is a map, not a string. The assertion is `recipe "loom" must name a profile`, so it pins the defaults-shape check and does not pin a rule about profile names.
- A default naming an unknown profile or a profile without the recipe's block: two `TestLoad` rows. `TestDispatchSetProfiles` repeats the unknown-profile case through the write path.
- `Profiles.Settings` refusals (unknown profile, no block for the recipe, with the covered recipes and a way forward): `TestSettings`.
- `Profiles.Default` (set and unset): `TestSettings`.
- Seed-Child writes only the task's recipe block: `TestPrepareRunSettings` rows "the task's settings lie over the loom block…" and "the same profile writes its darn block for a darn task" give the loom/darn split for one profile.
- Task with no profile takes its recipe's default: `TestPrepareRunSettings` row "no profile takes the recipe's default profile".
- Task with no profile and no default for the recipe writes only its own settings: `TestPrepareRunSettings` row "no profile and no default for the recipe…".
- Template default applied when no profiles file exists: `TestPrepareRunSettings` row "no profiles file takes the template's default for the recipe". It covers only the darn default; the loom default `solid` has no such row.
- Profile without the recipe's block routes to Stuck: `TestPrepareRunSettings` row "a profile without the recipe's block is Stuck". `TestSeedChild_RunSettings` already covers `ErrInvalidRunSettings` → Stuck.
- Seed-Child passes the resolved recipe (board type, default `loom`): `TestSeedChild_RunSettings`, `wantRecipe` on the first two rows. The remaining rows leave `wantRecipe` empty, so nothing is asserted there.
- `lyx board profiles` prints the defaults and the new profile names: `stepProfiles`.
- Board upsert checks only the profile name: the `stepRunSettings` unknown-profile and `solid` rows.
- The configreg profiles check names the profile and recipe and refuses an unknown key or module in a block: `TestProfilesCheckLoad`, two rows.
- Retired `darn.review_*` / `darn_*` keys and the removed `budget`, `darn-wide`: no test asserts their absence. `TestLoad` and `TestTemplateLadder` pin the new names only. This is not a finding, since an absence assertion would be one more redundant test.

**Findings**

1. LOW, scope, `internal/runprofile/runprofile_test.go` (`TestTemplateLadder`, around line 100) and `template.yaml:35-38`: the `docs` profile's content is pinned by no test (evidence above). The board entry's operator decision gives it a Sonnet writer at medium effort and one Code perch round, which makes it a behaviour the diff adds with no coverage. Fix: add a `docs` row or assertion to `TestTemplateLadder` or `TestLoad`: darn block `darn.writer == "sonnet[medium]"`, `loom.code_max_bounces` 1 and hard cap 2, and no loom block, e.g. `Settings("docs", "loom")` errors.

2. LOW, scope, `internal/battencli/runsettings.go:20-26` against `runsettings_test.go`: `prepareRunSettings` now loads the profiles file before checking whether the task names a profile. A task with no profile and no default for its recipe now fails Stuck on an undecodable `profiles.yaml`. Before the diff, such a task never read the file. Every `ErrInvalid` row in `TestPrepareRunSettings` names a profile, and the "no profile and no default" row uses a valid file, so nothing pins this path in either direction. The decision record lists the refusal of an old file as an open risk, so it looks intended, but it is unpinned. Fix: add a row with `Recipe: "loom"`, no profile, `profilesYML: "light: 3\n"` and `wantStuck: ["profile \"light\""]`, or decide and document that a task without a profile ignores a broken file.

**Test economy**

- No new test duplicates another's coverage in a way that justifies a finding. The `TestDispatchSetProfiles` row "a default naming an unknown profile is refused" overlaps the `TestLoad` row, but it pins the config write boundary (file unchanged, exit code 1), so I don't count it as redundant.
- `TestSeedChild_RunSettings` and `TestPrepareRunSettings` test different layers, so there is no duplication.
- `TestTemplateLadder` repeats the ladder table as literals. That is the behaviour under test, not a restated stencil.
- Retired `merge_max_bounces` is still in `battencli/runsettings_test.go` (profile `light`, dropped-key assertion) and `configreg/runsettings_test.go` ("retired webster_max_bounces and merge_max_bounces passes"). These rows pin retired-key migration and were not added by this diff. I report them so the handler can judge the focus item that says no `merge_max_bounces` should remain; I did not file them as findings.

**Tool calls**

1. `git diff 04aafa88c..HEAD` over the five test files (`runprofile_test.go`, `battencli/runsettings_test.go`, `boardcli/verbs_test.go`, `configreg/runsettings_test.go`, `configcli/configcli_test.go`): my context held only the non-test diffs.
