---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: internal/boardcli/doc.go:66
    summary: boardcli doc still states the old profiles output shape and omits the defaults field
    fix: In internal/boardcli/doc.go line 66, change the documented output to `{"ok":true,"defaults":{…},"profiles":[{"name":…,"settings":{recipe:{module:{…}}}}]}` and add that defaults maps each recipe to the profile a task of that recipe takes when it names none.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/boardcli/cli.go:487
    summary: upsert help for the profile field does not say an empty profile takes the recipe's default
    fix: In internal/boardcli/cli.go line 487, extend the profile field description to say that an empty profile takes the task's recipe's default profile.
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/runprofile/runprofile_test.go:100
    summary: the darn-only docs profile's content is pinned by no test
    fix: In TestTemplateLadder or TestLoad in internal/runprofile/runprofile_test.go, add a docs case asserting the darn block holds darn.writer sonnet[medium], loom.code_max_bounces 1 and code_hard_max_bounces 2, and that Settings("docs", "loom") errors.
  - id: F4
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/battencli/runsettings_test.go
    summary: prepareRunSettings now loads profiles.yaml even for a task naming no profile, and no row pins that an undecodable file sends such a task to Stuck
    fix: In TestPrepareRunSettings add a row with Recipe loom, no profile and an old-shape profiles file where profile light maps to the number 3, expecting the Stuck refusal naming the profile.
  - id: F5
    severity: NIT
    class: consistency
    origin: handler
    location: internal/runprofile/runprofile.go:174
    summary: decodeDefaults doc claims it refuses a profile named defaults, but decode skips that entry before any profile check
    fix: In the decodeDefaults doc comment in internal/runprofile/runprofile.go, replace the sentence about a profile named DefaultsEntry with one saying the defaults entry is never read as a profile, since decode parses it only through this function.
---

### [MEDIUM] boardcli doc keeps the old profiles output shape

**Location:** internal/boardcli/doc.go:66
**Issue:** The diff added a `defaults` field and nested `settings` per recipe in `lyx board profiles`, and updated the help text, but the module doc still shows the old envelope. CLAUDE.md requires the module doc to land in the same commit.

### [LOW] upsert help omits the default-profile behaviour

**Location:** internal/boardcli/cli.go:487
**Issue:** The `profile` field description says only that the name is laid under run_settings and that "" clears it; the new behaviour, that an empty profile takes the recipe's default, is documented elsewhere but not here.

### [LOW] docs profile content untested

**Location:** internal/runprofile/runprofile_test.go:100
**Issue:** `TestLoad` checks only that `docs` exists and `TestTemplateLadder` skips it, so changing its writer or Code round breaks no test, though the board entry fixes both.

### [LOW] Profile-less task now reads profiles.yaml, unpinned

**Location:** internal/battencli/runsettings_test.go
**Issue:** Before the diff a task without a profile never read the file; now a broken file sends it to Stuck. The decision record lists this as an open risk, so it looks intended, but no row pins it.

### [NIT] Misleading decodeDefaults comment

**Location:** internal/runprofile/runprofile.go:174
**Issue:** The comment says a profile named `defaults` is refused here, but `decode` skips that name in the profile loop, so the entry is only ever parsed as the defaults mapping.

## Rejected

- Retired `merge_max_bounces` in `configcli_integration_test.go`, `configreg/runsettings_test.go` and `battencli/runsettings_test.go` (lens:code-tests, lens:code-pattern-fit): these are retired-key migration fixtures, not references to a live key, and the focus item's grep for leftovers is satisfied.
- Loom default `solid` has no no-file Seed-Child row (lens:code-tests, noted in passing): `TestLoad` pins the template defaults and the darn row exercises the same path, so a second row would duplicate coverage.

Focus questions settled: the template matches the board table for all four rungs with Code 1 in every loom block and each hard cap its rounds plus one; `docs` is darn-only with a Sonnet medium writer and one Code round; defaults are loom `solid` and darn `light`; Seed-Child writes only the task's recipe block and a missing block, a missing default and the reserved entry are all handled; speed's 0 rounds turns the perch off through profile settings, as the decision record states; the commit message claims hold.
