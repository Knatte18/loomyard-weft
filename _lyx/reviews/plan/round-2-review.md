---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/03-verify-nested-module.md:45
    summary: The test row cd nested && go -C ../other test ./pkg is planned to fire, but card 3's own rule makes it silent, and no row tells -C resolved against a cd apart from -C resolved against the root
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/05-caller-scan-types.md:51
    summary: Card 5 does not state how typed references and scan matches for one subject and file merge, so D4's one finding per subject and file and the card's claim that a typed reference is never counted twice both fail when a file or line holds both
  - id: F3
    severity: NIT
    class: scope
    location: _lyx/plan/04-caller-scan-import-path.md:30
    summary: Card 4 replaces the reference rules that scanReferences' doc comment restates, but does not target scanReferences, so the comment goes stale until card 5 or is edited outside the card's targets
---

### [MEDIUM] A verify-nested-module test row contradicts the check's own rule

**Location:** `_lyx/plan/03-verify-nested-module.md:45` (the Tests paragraph), against line 31 of the same card.

**Issue:** The planned row `cd nested && go -C ../other test ./pkg` is expected to fire "when `other` is another nested module".
Under the card's own rule, `-C ../other` resolved against the `cd` puts the command's directory at `other`, and `./pkg` then resolves to `other/pkg`.
`ModuleOf` returns `other` for both, so the command is reported only when "that module is nested (not `.`) and differs from `ModuleOf` of the command's directory" (line 31), which it does not.
The row is silent by construction.
The implementer has to either fail the test or bend the rule to match it.
The row is also supposed to show that `-C` resolves against a preceding `cd` (D8), but it can't: resolving `../other` against the root leaves the root, so the row gives no verdict that tells the two resolutions apart.

**Fix:** Replace the row with one whose verdict depends on resolving `-C` against the `cd`.
For example, `cd nested && go -C .. test ./nested/pkg` fires: the command's directory is the root, and the package `nested/pkg` is in module `nested`.
If the row is also meant to cover a second nested module, keep `cd nested && go -C ../other test ./pkg` as a silent row.

### [LOW] Card 5 leaves the merge of typed references and scan matches unstated

**Location:** `_lyx/plan/05-caller-scan-types.md:51–54` (the `typedReferences` paragraphs, the re-signed `scanReferences` and the filter on `unresolved` lines).

**Issue:** `typedReferences` returns `references` per subject, and the re-signed `scanReferences(root, files, subjects)` returns its own per-subject map.
In a checked file, scan matches are then kept "only at a line `unresolved` records".
The card does not say which function applies that filter, since `scanReferences` takes no `unresolved` argument.
It also does not say that the two maps are merged before reporting.
For a package-level subject, both maps feed blocking `uncoveredFinding`s, one per subject and file, so a checked file holding a typed reference on one line and an unresolved one on another becomes two findings for one subject and file unless the maps are merged.
That breaks D4's "one finding per subject and file as today".
For a method subject, a line holding both a typed `t.Method()` and an unresolved `undefined().Method()` is reported twice: blocking, and inside the collapsed informational finding.
That contradicts the card's claim (line 54) that "a typed reference is never counted twice".

**Fix:** State in the Intent that `callerCoverageFindings` drops each checked file's scan matches outside that subject's `unresolved` lines.
Also state that it merges a package-level subject's typed and scan lines per file into one `uncoveredFinding`.
For a method subject, state that it drops a scan match at a line `typedReferences` already counts.

### [NIT] Card 4 leaves scanReferences' doc comment stating the deleted rules

**Location:** `_lyx/plan/04-caller-scan-import-path.md:30–47` (Intent), against `internal/planglyph/callercoverage.go:138–143`.

**Issue:** `scanReferences`' doc comment restates today's matching rules: "elsewhere as the declaring package clause, a dot and the identifier" and "a method subject is referenced by a dot followed by its identifier, anywhere".
Card 4 replaces those rules with `referencesSubject`'s import-path rule and the import-name exclusion.
It names `callercoverage.go`'s file comment and `doc.go` for updating, but neither targets `scanReferences` nor mentions its comment.
Until card 5 re-signs `scanReferences`, its comment describes rules that no longer hold.
The only way to fix the comment on card 4 is a change outside that card's targets.

**Fix:** Add `internal/planglyph#scanReferences` to card 4's `Edit:` group, with no arrow.
In the docs sentence, say that its doc comment points the matching rule at the new resolution instead of restating the old one.

## Focus verdicts

- Card 5's testing seam: not a defect.
  `internal/webstercli/cli_test.go`'s `TestValidateCmd_Envelopes` runs `planglyph.Validate` over `deleteOrderPlan(1, 2)` and `(2, 1)` in `plankit.Repo(t, deleteOrderFiles)`.
  That tree holds only `sub/a.go` and `sub/b.go` under a fresh `t.TempDir()`, with no `go.mod`, so card 5's no-`go.mod` skip keeps the load from being called.
  I checked every other untagged test that reaches `planglyph.Validate`, `index.ValidateFormat`, `index.ValidateRework` or the `loomshed` plan gates: `loomshed`, `loomcli`, `loomrecipe`, `shedrecipe`, `loomengine`, `webstercli` and `websterengine`.
  None writes or copies a `go.mod` into its worktree root.
  The only untagged test outside `planglyph` that writes a `go.mod` is `websterengine/verifygate_test.go`, and it reaches no plan gate.
  `planglyph`'s own untagged tests are covered by the `TestMain` replacement.
  The card's targets need no other test file.
- Card 5's unresolved-identifier path: the card states how such an identifier reaches the scan (`unresolved` recorded per subject, scan re-run over the checked file, matches kept only at recorded lines).
  That keeps the file's other name matches from being re-reported.
  The remaining gap, how the typed and scan results combine, is F2.
- Card 4's import-name exclusion: not a defect.
  The Intent states what the exclusion lets through: a method call on a local that shadows an imported package name, which is then not matched even informationally.
  It also states the bound: the exclusion applies only in the scan, so after card 5 only in a file or at an identifier the load left without type information, where card 5's typed path resolves the call through its receiver.
  In Go, a file-scope import name can be shadowed only by a local, since the package block may not redeclare it, so the stated case is the whole of what the exclusion lets through.
