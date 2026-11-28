---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/runprofile/template.yaml:22
    origin: lens:code-correctness
    summary: On an existing hub whose operator repointed defaults.arras to another starting profile, the reconcile keeps that default but drops every arras block, and commits a profiles.yaml that runprofile.Load refuses, which breaks profile loading for every recipe
    fix: "Ship a one-time profiles reinterpretation, run by hubreconcile, that drops `defaults.arras` from the hub's profiles.yaml when the profile it names holds no arras block. Give it a new ID constant and a Rewrite func, and register it in the profiles module's `Reinterpretations` list at internal/configreg/configreg.go:171, beside the existing `dobbyengine.ProfilePullRequestModesReinterpretationID` entry. Have the Rewrite return a line naming the dropped default. Add a test row for the upgrade path. Reconcile a hub file and template base holding the pre-change template with `defaults.arras` set to `light`, through the hub reconcile walk or ReconcileHubWideAt followed by the new Rewrite. Assert that `runprofile.Load` over the result succeeds and that `Defaults` holds no arras entry."
  - id: F2
    severity: LOW
    class: consistency
    location: _lym/landing/summary.md#Merge danger
    origin: lens:code-affected-set
    summary: The blast radius names only runprofile, but the template change rewrites every existing hub's committed profiles.yaml and template base through configsync and hubreconcile, and the claim that an operator-changed arras key stays omits that a kept default can outlive the block it names
    fix: "In _lym/landing/summary.md, widen `**Blast radius:**` to name the hub reconcile (configsync, hubreconcile) and each existing hub's committed profiles.yaml and profiles.template-base. In the `## What changes` bullet on the reconcile, say how a `defaults.arras` the operator changed is handled once its profile's arras block is dropped, as the F1 fix settles it."
  - id: F3
    severity: NIT
    class: consistency
    location: internal/runprofile/runprofile_test.go:125
    origin: lens:code-tests
    summary: The new Default(arras) assertion in TestTemplateLadder duplicates TestLoad's row "no file reads the template's ladder, docs and defaults", which already checks the template's defaults by exact map equality and fails if an arras default returns
    fix: "Delete the `if got, err := profiles.Default(\"arras\"); ...` block at internal/runprofile/runprofile_test.go:125-127. Change the doc comment line above TestTemplateLadder to state only that no profile holds an arras block."
---

### [BLOCKING] The reconcile can commit a profiles.yaml that no longer loads

**Location:** `internal/runprofile/template.yaml:22`; the reconcile path is `internal/yamlengine/follow.go:155-187`, `internal/configsync/configsync.go:571-577`, `internal/hubreconcile/walk.go:250-265` and `internal/configcli/hubwide.go:224-232`.

**Issue:** `FollowTemplate` judges each unit against the template base on its own.
Take a hub whose operator set `defaults.arras: light` and left the starting profiles' arras blocks as shipped.
The reconcile keeps `defaults.arras`, since it differs from the base's `solid`.
It removes `light.arras`, since that block equals the base and the new template lacks it.
The written file holds `defaults: {arras: light}` with no `light.arras` block.
`decodeDefaults` refuses that file (`internal/runprofile/runprofile.go:251-253`), so `Load` fails with `ErrInvalid` for the whole file.
Neither reconcile caller runs the module's `CheckLoad` after writing, so the broken file is committed.
From then on every consumer that loads profiles fails for every recipe, not only arras:

- Seed-Child's `prepareRunSettings` (`internal/battencli/runsettings.go:26-29`);
- the board's profile validation (`internal/boardcli/runsettings.go:47-50`);
- the raddle context (`internal/loomcli/raddlecontext.go:58-61`);
- `checkProfiles` (`internal/configreg/runsettings.go:231-235`).

PATTERN-config-layers says a meaning change ships a reinterpretation, run by `hubreconcile`.
No test row covers this case.
The other cases on the focus's upgrade path load cleanly:

- an untouched file;
- an operator-edited arras block, which leaves a partial block that still decodes;
- an operator-added profile with its own arras block named by `defaults.arras`;
- a file with no template base.

The code-tests lens filed the missing test row as a separate MEDIUM finding; it is folded into this one.

### [LOW] Merge danger names less than the change reaches

**Location:** `_lym/landing/summary.md`, `## Merge danger`.

**Issue:** `**Blast radius:** runprofile` leaves out the main way the change reaches a hub.
On its next reconcile, every existing hub's committed `profiles.yaml` and `profiles.template-base` are rewritten through `configsync` and `hubreconcile`.
In F1's case that rewrite breaks profile loading across the hub.
The `two-way` door holds by itself: reverting the template makes the follow add the blocks back at the template's values.
The summary's reconcile bullet, "an arras key the operator changed stays, as for any other template change", holds key by key.
It does not say that a kept default can name a block the reconcile dropped.

### [NIT] Duplicate arras-default assertion

**Location:** `internal/runprofile/runprofile_test.go:125-127`.

**Issue:** `TestLoad`'s row "no file reads the template's ladder, docs and defaults" compares the template's `Defaults` with `maps.Equal` against a literal (`runprofile_test.go:48`, `:90`).
That comparison already fails if an arras default returns, so the new `Default("arras")` assertion adds no coverage.

## Rejected

- Code-tests lens, MEDIUM "upgrade path with an operator-repointed default has no test": the same defect as F1, folded into F1's fix.
