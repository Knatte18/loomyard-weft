---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:code-pattern-fit
    location: internal/shuttleengine/claudeengine/doc.go:237
    summary: The open-launch paragraph still says the bypass flag is carried only when the open's permission mode is bypass, but an empty mode now carries it too.
    fix: In internal/shuttleengine/claudeengine/doc.go line 237, reword the sentence to "It carries `--dangerously-skip-permissions` unless the open's permission mode is `prompt`, on the fresh and the resume line alike, and an unknown mode refuses the open."
  - id: F2
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/shuttlecli/open.go:56
    summary: The --list-models branch's registry-load failure (error envelope naming --list-models) has no test.
    fix: In internal/shuttlecli/cli_test.go, add a case to TestOpenCmd_ListModels (or a sibling row) that writes an unparsable models.yaml under configengine.ConfigFile(anchor, "models"), runs --list-models, and asserts an error envelope (ok false) whose error contains "--list-models".
---

### [MEDIUM] Stale open-launch bypass description in the claudeengine package doc

**Location:** internal/shuttleengine/claudeengine/doc.go:237
**Issue:** The diff makes an empty permission mode resolve to bypass for opened sessions, and the board entry and decision record list the docs this change touches. `claudeengine/doc.go` still states the flag is carried "when the open's permission mode is `bypass`", so a reader concludes a default open does not bypass. This breaks CLAUDE.md's "docs land in the same commit" rule for the module doc. Confirmed by the affected-set and pattern-fit lenses.

### [LOW] No test for the --list-models registry-load failure

**Location:** internal/shuttlecli/open.go:56
**Issue:** The new `LoadRegistry` error branch under `--list-models` returns an error envelope and opens nothing, but no test reaches it. `TestOpenCmd_ListModels` covers only the success path.

## Rejected

- Tests lens F1 (no assertion of the bypass flag on the open launch's fresh line): the resume line embeds the fresh line as its fallback, so the `TestOpenLaunch` assertion of two flag occurrences in `ResumeCmd` already requires the flag on `Cmd`.
- Tests lens F2 (hard-coded count of 2): the same count is already the convention in the neighbouring `bypass skips permissions on every line` case, so this change adds no inconsistency.
- Correctness lens: reported no defects; each changed branch ran correct, including `--list-models` with no `--role`, an empty registry, a missing default effort and the `opus[high]` envelope.
