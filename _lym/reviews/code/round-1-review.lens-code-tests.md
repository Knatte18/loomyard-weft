```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I looked for the test that pins it.

**Per behaviour, the test that pins it**

1. **The template no longer seeds `defaults.arras`.** Pinned three times:
   - `TestLoad`, row "no file reads the template's ladder, docs and defaults", checks the defaults with an exact `maps.Equal` (`internal/runprofile/runprofile_test.go:48`, compared at `:90`).
   - The new `Default("arras")` assertion in `TestTemplateLadder` (`internal/runprofile/runprofile_test.go:125-127`).
   - `stepProfiles` checks the defaults with an exact `DeepEqual` (`internal/boardcli/verbs_test.go:1077`).
2. **No ladder profile holds an `arras` block.** Pinned by `TestTemplateLadder`, which asserts `Settings(profile, "arras")` errors for each ladder profile (`internal/runprofile/runprofile_test.go:189-191`).
3. **An arras task that names a starting profile is refused at Seed-Child.** Pinned in the runprofile layer by the same `Settings` error assertion. The batten refusal path is the existing generic row "holds no motif block" (`internal/battencli/runsettings_test.go:~57`).
4. **An arras run naming no profile runs on the hub's config.** Pinned by `Default("arras") == ""` plus the existing batten row "no profile and no default for the recipe writes the task's settings alone" (`internal/battencli/runsettings_test.go:66`).
5. **The reconcile drops the arras keys from an untouched hub file still at the template's old values.** Pinned by `TestReconcileHubWideAt_SeedOnlyFileFollowsItsTemplate`, row "the untouched file takes the new template". It compares the whole reconciled file with the current template, and the fixture `testdata/profiles-flat.yaml` still holds `defaults.arras` and the four arras blocks (`internal/configsync/configsync_test.go:~873`; fixture lines 26, 41, 58, 75, 92).
6. **The reconcile with an operator-edited arras key.** No test. This covers two cases:
   - The operator repointed `defaults.arras` to another starting profile and left the arras blocks untouched.
   - The operator edited one arras block.

   Nothing pins what that file loads as afterwards.
7. **Doc and glossary prose.** Exempt as static prose.

**Findings**

- **MEDIUM, design.** `internal/runprofile/template.yaml:22` (the removed `arras: solid`). The upgrade path where the operator set `defaults.arras` to another profile and left the arras blocks untouched has no test. If such a test existed, it would fail.
  - `FollowTemplate` keeps the operator-set scalar `defaults.arras`, because the file differs from the base (`internal/yamlengine/follow.go:166-170`).
  - It removes the base-valued block `&lt;profile&gt;.arras` (`follow.go:176-178`).
  - `decodeDefaults` then refuses the file, because the default names a profile with no arras block (`internal/runprofile/runprofile.go:251-252`).
  - Neither the hub reconcile nor `lym config reconcile` runs `CheckLoad` after writing (`internal/hubreconcile/walk.go:250-265`, `internal/configcli/hubwide.go:224-232`).
  - The whole `profiles.yaml` then fails `Load`. That breaks every recipe's Seed-Child (`internal/battencli/runsettings.go:26-29`), board profile writes (`internal/boardcli/runsettings.go:48-51`) and raddle context (`internal/loomcli/raddlecontext.go:58-61`).
  - The closest test, `TestReconcileHubWideAt_SeedOnlyFileFollowsItsTemplate` (`internal/configsync/configsync_test.go:839-1140`), has no row where the reconciled profiles file is then loaded.
  - It needs a row that reconciles a hub file holding `defaults.arras: light` against the pre-change base and asserts the result passes `runprofile.Load`. That row would fail today, so the code needs fixing too.
- **NIT, consistency.** The new `Default("arras")` assertion at `internal/runprofile/runprofile_test.go:125-127` duplicates coverage. `TestLoad`'s row "no file reads the template's ladder, docs and defaults" already checks the template defaults by exact map equality (`runprofile_test.go:48`, `:90`), and that check fails if an arras default comes back.

**Tool calls**

1. Grepped the battencli and runprofile/configsync tests for no-default and dangling-default rows: the manifest held no battencli test file.
2. Read the battencli prepare-run-settings test rows and grepped the runprofile load-refusal rows: needed to confirm the no-default path and the dangling-default refusal are covered.
