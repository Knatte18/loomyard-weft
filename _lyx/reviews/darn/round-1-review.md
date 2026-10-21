---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/configengine/config.go:207
    summary: The new filter that drops per-name map keys from the filled-keys Info log has no test.
    fix: Add a row or assertion to the existing log test at internal/configengine/config_test.go:93, using a Spec with PerNameMaps. A base file lacking a name under a declared per-name map must not log that key, and a base file lacking the per-name map path itself must still log that path.
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: consistency
    location: internal/configreg/configreg.go:49
    summary: The PerNameMaps field comment still says configsync adds the template's entries per name, which reconcile no longer does.
    fix: In internal/configreg/configreg.go, reword the PerNameMaps comment (lines 49-51) to say configsync keeps only the names the file changed, leaves out a name holding the template's value, never copies a template name in, and keeps every entry the template lacks.
    origin: handler
  - id: F3
    severity: LOW
    class: consistency
    location: internal/configcli/configcli.go:627
    summary: The `lyx config reconcile` help text omits the new `dropped` result key and that --apply also rewrites on a drop.
    fix: In the reconcile command's Long text (internal/configcli/configcli.go:627-632), add a sentence that a per-name map entry holding the template's value is reported under "dropped" and left out of the written file. Extend the --apply sentence at line 632 so it says the file is rewritten when the reconcile adds, removes or drops anything.
    origin: lens:code-affected-set
---

### [MEDIUM] Filled-keys log filter is untested

**Location:** internal/configengine/config.go:207
**Issue:** The diff adds a `slices.DeleteFunc` filter so a filled key under a declared per-name map is not logged, while a filled per-name map path still is. No existing log test declares `PerNameMaps`, so none fails if the filter is removed or over-drops. This is a behavior the diff adds with no coverage, under `PATTERN-test-economy`.

### [LOW] Stale PerNameMaps field comment

**Location:** internal/configreg/configreg.go:49
**Issue:** The comment says "configsync adds the template's entries per name and keeps every entry the template lacks". Reconcile now never copies a template name in and drops a template-equal one.

### [LOW] Reconcile help text lacks `dropped`

**Location:** internal/configcli/configcli.go:627
**Issue:** The command's long help describes reported keys without `dropped`, and its `--apply` sentence is now incomplete, since a drop-only change rewrites the file.

## Rejected

- The `dropped` attribute added to the hub reconcile walk's log line has no test (lens:code-tests, LOW): a log-only attribute that decides nothing, and the summary's claim holds against `walk.go:366`.
- Every code-correctness case the fork ran (absent, null and empty per-name map, style-insensitive equality, `SetValues` including a name a pair set to its template value, both `configsync` sites, the sparse-layer path) held, so it raised no finding.
- Pattern-fit found no rule broken, and the focus's docs, summary-claim and sparse-layer questions all settled as correct.
