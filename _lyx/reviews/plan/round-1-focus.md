---
round: 1
focus:
  - "Card 1, `TestRunner_Open`: the plan drops `t.Parallel` on the whole test to capture the process-global logger. Does an existing test or helper already capture or assert that log line, so the new subtest assertions duplicate coverage?"
  - "Card 2 against `TestValidatePermissionMode` and the existing `TestOpenLaunch` rows: do the new `bypass`, `prompt` and `auto` rows re-cover a branch an existing row already pins, beyond the `ResumeCmd` shape the plan names?"
  - "Card 3, `TestOpenCmd_ModelAliasEngine`: does converting both engines to `openRecorderEngine` change what the existing alias row pins? Is the `openRecorderEngine` type and its `OpenLaunch` method one granularity unit, or does the plan split what belongs together?"
  - "Fidelity to the decision record: is each of D1 to D5, the constraints and AC1 to AC6 carried by some card, including the `buildOpenCmd` and `Claude.OpenLaunch` doc comments, the `claudeengine` and `shuttleengine` package docs and the `docs/overview.md` shuttle row? Does any card add scope the record excludes?"
  - "Attack-surface bound: the bypass edge is stated in Card 2 and Card 3 `Intent`. Does each state what it lets through and what bounds it consistently with D5? Is the Card 1 refusal-before-creation claim reachable by a named input and a named test?"
  - "Verify coverage: does `## verify:` run the tests of every package the cards target, including `internal/shuttleengine/claudeengine`, `internal/shuttlecli` and `cmd/lyx`, and does the section follow the stencil's order?"
  - "Card 2 `buildOpenCmd` re-sign arrow and member glyphs: does each `Edit` glyph whose parameters, results or receiver change carry the arrow, and do the Go-file targets in Card 1 and Card 3 use member glyphs for the symbols the `Intent` names?"
---

Seed focus from a read of the overview, the three cards and the decision record.
The plan is small and ordered 1, 2, 3 with no backward dependency visible.
`support-log.md` was not read.
