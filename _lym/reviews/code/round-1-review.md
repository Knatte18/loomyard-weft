---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: internal/reedcli/window.go:80
    summary: lym reed window help text still describes the old title forms
    fix: >-
      In the Long text of the window command in internal/reedcli/window.go, replace the three old forms on lines 81-83
      with the new ones.
      A task reads <T#> [<hub shortname>:<P#>] <slug>.
      The prime's and a beam's windows read ORCH [<hub shortname>:<P#>] <slug>, and the prime's names the root project, as ORCH [<hub shortname>:P0] root.
      Keep the sentence that a part that does not exist is left out.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/sharedbootstrap.go:380
    summary: statusWindowTitle doc comment still shows the old title example and speaks of the prime's label
    fix: >-
      In the doc comment of statusWindowTitle, change the example ly: P3: T4: tapestry to T4 [ly:P3] tapestry,
      and replace the sentence that the prime's own label is never used with wording that no prime name is passed,
      so the prime is never titled as the orch.
  - id: F3
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/reedcli/window_test.go:27
    summary: stub title constant still uses the old format
    fix: >-
      Change formedTitle in TestWindowCmd_OpensATerminalAttachedToTheToldWorktree
      from "ly: P3: T7: task" to "T7 [ly:P3] task".
  - id: F4
    severity: NIT
    class: design
    origin: lens:code-correctness
    location: internal/consoletitle/consoletitle.go:371
    summary: Title appends an empty name, leaving a trailing space although the doc says an empty part is left out
    fix: >-
      In Title, append name to parts only when it is non-empty before joining,
      so Title("ORCH", "ly", "P0", "") reads "ORCH [ly:P0]", and add one TestTitle row for an empty name.
---

### [MEDIUM] reed window help text still shows the old forms

**Location:** internal/reedcli/window.go:80
**Issue:** The `lym reed window` Long text says the default title is `<hub shortname>: <P#>: <T#>: <slug>`, `<hub shortname>: orch` for the prime and `<hub shortname>: <P#>: orch` for a beam.
The title now comes from `ConsoleWindowTitle`, which builds the bracket form, so the operator-facing help describes a form no window carries.
The focus directive names `reedcli/window.go` as a place to look for exactly this.

### [LOW] statusWindowTitle doc comment keeps the old example

**Location:** internal/loomcli/sharedbootstrap.go:380
**Issue:** The comment gives `ly: P3: T4: tapestry` as the example and says the prime's own label is never used.
The label constants are gone and the example is the old form.

### [NIT] stub title constant in the window test is the old shape

**Location:** internal/reedcli/window_test.go:27
**Issue:** `formedTitle` is a stubbed value the test passes through, so no assertion fails, but it is the one remaining old-format string in the tests.

### [NIT] Title keeps a trailing separator for an empty name

**Location:** internal/consoletitle/consoletitle.go:371
**Issue:** `Title` appends `name` even when it is empty, so `Title("ORCH", "ly", "P0", "")` returns `ORCH [ly:P0] `.
The doc comment says an empty part is left out with its separator.
No current caller passes an empty name, so this is a boundary slip and not a live defect.

## Rejected

- Slack channel names not changed in this diff (lens code-affected-set, MEDIUM scope): the `slack-per-orch-activation` entry's own body already specifies the new channel names, such as `osl46-mo-p0-root`, and limits its change to `internal/slackengine`, `internal/slackcli`, reed's status bar and the orch role file. The Goal's sentence cross-references that entry, so the diff has no Slack change to make.
- Circular `wantName` in `smoke_drivingsurface_test.go` (lens code-tests, NIT): the test already built its expectation with `consoletitle.Title` before this diff, only the arguments changed, and `TestRecipeWindowTitle` pins the shape with literals.
- The `T10 [ly] orphan` project-missing case (lens code-correctness, noted as a design choice): it matches the change description's decision and is pinned by `ATaskWhoseProjectIsMissingLeavesOutTheProject`; the Goal does not rule it out.
- Test economy: the redundancy report lists no test as contained, and `TestTitle` and `TestWindowTitles` pin different layers, so no test-economy finding stands.
