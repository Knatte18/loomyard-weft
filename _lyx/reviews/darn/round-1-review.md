---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:code-pattern-fit
    location: internal/runprofile/runprofile.go:586
    summary: runprofile.Load adopts neither configengine.Load nor LoadOrTemplate, which PATTERN-config-strictness says a caller adopts exactly one of, and PATTERN is not updated
    fix: In pattern/PATTERN-config-strictness.md add runprofile to the caller lists as a third stance, one that reads the stacked layers through configengine.StackLayers with no template fill and falls back to the template only when no layer file exists, and state the reason (a removed starting profile must stay removed). Adjust the one-line entry in PATTERN.md so it no longer says exactly one of the two. Alternatively load through one of the two policies, if that keeps a deleted starting profile deleted.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/boardcli/verbs_test.go:518
    summary: No test exercises the profile check on upsert-batch or merge, so dropping the prepareRunSettings call on either path passes every test
    fix: Extend the existing run-settings tests in verbs_test.go (stepRunSettings or its batch and merge counterparts) with one unknown-profile row for upsert-batch and one for merge, each expecting the refusal that names the known profiles.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/boardcli/runsettings.go:231
    summary: The profile-specific running-pair notice and the unchanged-profile re-write from a task session are untested
    fix: Add one row to the existing notice test and one to the task-session test in verbs_test.go. The first changes the profile of a task whose stored status is past Seed-Child and expects the notice. The second re-sends the stored profile from a task-slug strand and expects no refusal.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/configreg/runsettings.go:448
    summary: The refusal of the merged document after layering ("layered over run profile") has no test row
    fix: Add a row to TestLayerRunSettings in internal/configreg/runsettings_test.go where the profile and the task settings each validate alone but the merged document is refused, and assert the RunSettingsError text names the profile.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/configengine/set.go:352
    summary: setEntries is tested only on the default layer, so its local-layer branch is unpinned
    fix: Add a --layer local row to TestDispatchSetProfiles in internal/configcli/configcli_test.go and assert the outcome the module's local rule gives, probably a refusal, with the file untouched.
---

### [BLOCKING] runprofile.Load is a third config loading policy and PATTERN does not say so

**Location:** `internal/runprofile/runprofile.go:586`
**Issue:** `PATTERN-config-strictness` says a caller adopts exactly one of strict `configengine.Load` or `LoadOrTemplate`, and names both caller sets.
`runprofile.Load` calls `StackLayers` directly, decodes the merged document without template fill, and reads the template only when no layer file is present.
That is a deliberate and sound choice, recorded in the decision record and `doc.go`, but the diff touches neither `PATTERN.md` nor `pattern/`.
`CLAUDE.md` puts a new cross-cutting invariant into PATTERN in the same commit.
The same note covers the direct reads of `profiles.yaml` by `boardcli` and `battencli`, which respect `PATTERN-hub-wide-config` and `PATTERN-config-layers` and need no separate fix.

### [MEDIUM] No test pins the profile check on upsert-batch or merge

**Location:** `internal/boardcli/verbs_test.go:518`
**Issue:** The decision record's acceptance criteria and the focus directive both require the check on all three upsert paths.
`prepareRunSettings` is called from each, but only `upsert` has profile rows, so a dropped call on `upsert-batch` or `merge` fails nothing.

### [LOW] Running-pair notice and unchanged profile are untested

**Location:** `internal/boardcli/runsettings.go:231`
**Issue:** The `changed` computation and the reworded notice for a profile change have no row.
The existing notice test covers a `run_settings` change only.

### [LOW] Layered-document refusal has no test row

**Location:** `internal/configreg/runsettings.go:448`
**Issue:** The second `checkLocalDocument` after `yamlengine.Merge` is the only guard against a pair of individually valid documents that merge into a refused one, and no row reaches it.

### [LOW] setEntries local-layer branch is untested

**Location:** `internal/configengine/set.go:352`
**Issue:** `TestDispatchSetProfiles` covers the default layer only, so the `ValidateLocal` call in `setEntries` is unpinned.

## Rejected

- Fork `code-affected-set` F1, `docs/shared-libs/configengine.md:19-24` names only `board` and `fabric` as hub-wide: the list already omitted `gate` and `language` before this diff, so the diff did not make it wrong.
- Fork `code-correctness` F1, an unchanged `profile` re-sent on a write is refused when the profile has since been removed: the board entry says an unknown profile name on a task is refused at the board write, and the refusal names the way forward.
- Fork `code-correctness` F2, `"profile": null` is refused where `run_settings: null` clears: the entry makes `""` the clearing value and `profile` a string field.
- Fork `code-tests` finding 5, no test shows the wire closure carries `task.Profile` into `PrepareRunSettings`: the fork itself withdrew it when no wire-level test exists, and none does.
- Shortened help pages of `lyx board upsert` and `lyx config`: the trimmed asides (`body`'s "(proposal / background)", `--body-file`'s "drop one of them" and "every other field still comes from the payload") are not facts an operator needs to run the verbs, and the 2000-byte cap in `cmd/lyx/clitree_test.go` is met.
- Scope against the board entry: every item of the amended entry is made (`profile` field, `profiles.yaml` hub module with the three starting profiles, seeding with `run_settings` on top, `lyx config profiles` with `--set` taking JSON, `lyx board profiles`, refusal of an unknown name at the board write and at seeding).
  `JSONEntries` and `yamlengine.SetEntries` are what the entry's `--set` clause needs, so they are licensed; no change outside the entry was found.
- Change description: each claim in `_lyx/landing/summary.md` that I checked against the diff holds.
