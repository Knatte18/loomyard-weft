```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change the diff makes, find what reads or asserts it outside the touched lines, and say whether it is covered.

**Affected set per change**

1. **`template.yaml` drops `defaults.arras` and the arras block of each starting profile.**
   - `runprofile.Load` and `decodeDefaults` (`internal/runprofile/runprofile.go:164-257`): covered for a fresh hub, since nothing names arras.
   - `configreg.checkProfiles` (`internal/configreg/runsettings.go:231`): covered.
   - `battencli.prepareRunSettings` (`internal/battencli/runsettings.go:24-57`): an arras task naming no profile now runs on the hub's config, and one naming a starting profile gets the `Settings` refusal. Covered, and this matches `doc.go`.
   - `boardcli.prepareRunSettings` (`internal/boardcli/runsettings.go:28`): only `Lookup` is called, so nothing changes. Covered.
   - `loomcli` raddle context (`internal/loomcli/raddlecontext.go:115-122`): reads only tapestry and darn coverage. Not affected.
   - `loomcli.writeRunUsage` (`internal/loomcli/sharedbootstrap.go:217-220`): an arras run records an empty default. Consistent.
   - Existing hub `profiles.yaml` reached through `configsync.ReconcileHubWideAt` and `yamlengine.FollowTemplate`: **not covered, finding 1.**
   - Tests: `TestLoad`, `TestTemplateLadder` and the boardcli `stepProfiles` are updated. The configsync key-move row "the untouched file takes the new template" (`internal/configsync/configsync_test.go:859-940`) compares the whole file to the new template, so it carries the drop. Covered.
   - Fixture `internal/configsync/testdata/profiles-flat.yaml`: it keeps its arras blocks as the pre-move template, which the summary states as a design decision. Covered.
   - The template's own header (`template.yaml:2-3`) names no arras block. Covered.
   - `contracts/recipes/arras-recipe.yaml:87,125`: these `profile:` keys are burler review profiles, not run profiles. Not affected.
2. **`runprofile/doc.go` adds the arras rule.** It matches `Default` returning empty and `prepareRunSettings` falling back to the hub's config. Covered.
3. **`docs/overview.md` and the glossary mark arras as kept but not offered.**
   - Each recipe list naming arras is marked, at `overview.md:334, 339, 1134-1135, 1981-1982`.
   - The other arras mentions are not recipe lists: `:1444, 1474, 1681` describe the split perch's config, and `:1713` already says arras is not offered.
   - The glossary row is updated.
   - Covered.
4. **Rules and records.**
   - PATTERN: no PATTERN entry or `pattern/` file names arras or profile defaults, and the directive declares no PATTERN changes. Nothing to land.
   - Commit messages: the range carries no `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer. None is needed: the removed arras assertions follow the template's removal and are replaced by the inverse assertion, and the task is not a bug fix.

**Findings**

1. **HIGH, `design`, `internal/runprofile/template.yaml:22`** (the dropped `defaults.arras`). On an existing hub, the reconcile can write a `profiles.yaml` that `runprofile.Load` refuses.
   - The follow judges each unit on its own (`internal/yamlengine/follow.go:155-187`).
   - A hub whose operator repointed `defaults.arras` to another starting profile, say `light`, keeps that default because it differs from the base. Every starting profile's arras block still equals the base, so each one is removed.
   - The result holds `defaults.arras: light` with no `light.arras`, and `decodeDefaults` refuses it (`internal/runprofile/runprofile.go:251-252`).
   - No load check runs after the reconcile write: `internal/hubreconcile/walk.go:250-265` and `internal/configcli/hubwide.go:224-232` call no `CheckLoad`.
   - `Load` then fails for every recipe: Seed-Child refuses every task (`internal/battencli/runsettings.go:25-28`), a board profile write fails (`internal/boardcli/runsettings.go:46-48`) and the raddle context fails (`internal/loomcli/raddlecontext.go:58-61`).
   - No test covers a template removal that strands an operator-kept default.
2. **LOW, `consistency`, `_lym/landing/summary.md` `## Merge danger`.** The blast radius names only `runprofile`, but the template change rewrites every existing hub's committed `profiles.yaml` and its template base through `configsync` and `hubreconcile`. In the case in finding 1, that breaks profile loading hub-wide. The summary's line "an arras key the operator changed stays, as for any other template change" does not mention that a kept default can outlive the block it names.

**Tool calls**

1. Grepped the `_test.go` files, stencils and docs for arras profile and default readers: the context held only the runprofile and boardcli tests.
2. Grepped the repo's `.md`, `.yaml` and `.tmpl` files and the Go sources for other arras and split-perch profile mentions, to find fixtures and contracts outside the manifest.
3. Grepped `contracts/recipes/arras-recipe.yaml`, `PATTERN.md` and `pattern/` for profile wording, to check whether the recipe contract or the PATTERN entries describe arras profiles.
4. Read `contracts/recipes/arras-recipe.yaml:75-135` to settle what its `profile:` keys are.
