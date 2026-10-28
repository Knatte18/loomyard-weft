```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked pairs of the record's own decisions, and each acceptance criterion and constraint, against the decisions they touch.

**Pairs checked**
- D1 × D2 (D2 reads the ids D1's headings give; a malformed heading gets a D1 finding and gives no id): held.
- D1 × D2's `lym loom decision add` carve-out (appended `### Added after Discussion` entries are exempt from D1 and D2): held.
- D2 × the existing early return in `ValidateDarnDiscussion` (D1 and D2 must run "beside Validate" in both callers, but the darn function returns early on Validate findings): broken. The ordering is unstated, see F2.
- D2 × D3/D4 (one shared D1/D2 function for loom and darn; citations are loom-only through `ValidateLoomDiscussion`'s new root and index): held.
- D3 × D4 token classes (a path excludes `#` and `:`; a glyph needs `#`; `pkg.Member` needs an upper-case member, so `recipe.go`, `gate.yaml` and `state.json` fall through untested): held.
- D3/D4 skips × the PATTERN changes suffix and the carry-over blocks Go writes (writer-uneditable text is skipped): held.
- D3/D4 × D2's own acceptance criteria (the `&lt;placeholder&gt;` rule, `_lym/…`, `internal/` names an existing directory, `os.ReadFile`, `C#`): held. This record's own criteria pass its own rules.
- D3/D4 × the PATTERN changes bullet of this record (`//lym:seam` holds `:`; `t.Parallel` has no package `t`): held.
- D4 × D10 (the spec tells darn writers the D3/D4 marks, but darn runs D1 and D2 only): see F4.
- D5 × the impactset guard scan (`onlyKeepLinesAfter` today refuses any non-keep line; D5 widens both parsers): held as decided.
- D5 × D6 (D6 consumes the markers; a new reflective test a card creates has no marker yet, so D6 is not held for it, and D5's tier0 scan catches it at code time): held.
- D5 × D9 (D9 has `PlanContext` carry the markers, but the record never says what builds them into it, see F3): partly broken.
- D5 × D11 (the stacking rule goes into the PATTERN background): held.
- D6 × D8 (the same later-card, Uses and package-self-glyph coverage pattern; avoids `uses-later-target` and `redundant-package-target`): held.
- D6/D8 × D9 done-card skip (`planContextFindings` gets pending cards, a later card is looked up in `plan.Cards`; rework passes all cards): held.
- D7 × D9 (no `gate.yaml` skips D7; a non-Go plan skips D6 to D8): held.
- D7 × D6 (a new test file is tier-checked; it carries no marker yet): held.
- D9 × the Constraints' gate/verb parity (`PlanContext` is built twice, in `validate.go:272` and in `planContextOf`, `entries_gate.go:126`): broken. The record does not say both builders load the tier tags and markers, see F3.
- D1 × Constraint `PATTERN-refusal-way-forward`: broken, see F1.
- D10 × D1 to D4 (scope of "loom and darn records"): ambiguous, see F4.
- D2 × `internal/loomshed/amendment.go` (writes no decision headings): held.
- Acceptance criteria × D1 to D11 (every id appears as a whole word): held. The D5 criterion misses one refusal D5 states, see F5.

**Findings**

F1 — LOW, consistency, decision-record.md D1.
D1 adds `decision-heading-malformed` and `decision-id-duplicate` with no way forward, while D2, D3, D4, D6, D7 and D8 each state theirs.
The Constraints bullet on `PATTERN-refusal-way-forward` requires every new finding to name its way forward.
D1 should state one, such as: rename the heading to `### D&lt;n&gt; — &lt;title&gt;`, or renumber the duplicate.
Evidence: decision-record.md D1 and its Constraints bullet.

F2 — LOW, design, decision-record.md D2, second paragraph.
D1 and D2 are said to run "beside" `discussionparser.Validate` in both callers, but their order relative to the existing checks is not stated.
`ValidateDarnDiscussion` returns early when Validate finds a defect (`internal/loomshed/gates.go:140-143`).
`ValidateLoomDiscussion` runs `CheckRelevant` even after Validate fails, so one re-prompt names every defect (`gates.go:119-131`).
Left unsaid, darn may hide D1/D2 findings behind a section finding and add a re-prompt that loom avoids.
The record should state whether D1 and D2 run when Validate finds defects, in both callers.

F3 — LOW, design, decision-record.md D9 against the gate/verb parity Constraint.
D9 says `PlanContext` carries the `gate.yaml` tier tags and the seam markers.
`PlanContext` is built separately by `validatePlanCmd` (`internal/loomcli/validate.go:265-277`) and by `planContextOf(env)` (`internal/shedrecipe/entries_gate.go:126-133`).
`Env` carries only the rendered `GateTiers` string and `RefuseRaw` (`recipe.go:115`, `wiring.go:406-411`).
Nothing in D5 or D9 says what scans the worktree for markers, or that both builders fill the new fields the same way.
A divergence breaks the parity the Constraint relies on without either gate noticing.
The record should name one function that both builders call to load the markers and tier tags.

F4 — NIT, consistency, decision-record.md D10.
D10 says the spec "states D1 to D4 for loom and darn records", which reads as if the citation marks and the `&lt;placeholder&gt;` rule apply to darn.
D4 runs D1 and D2 only on darn.
D10 should say D3 and D4 are loom-only in the spec.

F5 — LOW, consistency, decision-record.md Acceptance criteria, the D5 bullet.
D5 also refuses "a marker whose glyph does not name a struct", and names `VisibleFields` and `envkit.NilSeams` as triggers.
The criterion covers only `NumField` and the stacking order, so the marker-glyph refusal has no scenario.
The criterion should add one.

Tool calls:
1. `internal/loomshed/amendment.go` and the entry of `discussionparser.Validate`: to check whether the amendment path writes decision headings D1 would judge. The manifest did not include amendment.go.
