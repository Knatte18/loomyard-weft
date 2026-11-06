---
round: 1
focus:
  - "internal/shellguard/rows.go `pythonRefusal`/`pythonRoutes`: does the steer and the notice (and the Codex steer) state the reason, the way forward to the operator or parent, and never working round the guard? Settle whether the sentence still reads correctly where `pythonRoutes` prepends `otherwise` and where the notice reuses the text."
  - "internal/shellguard/rows.go init check: confirm the new text contains no quote or backslash, including the `: false` form from `SwitchDenyPython`, and that the Codex steer (which swaps the edit rule) still carries the reason."
  - "internal/configcli/configcli.go `buildConfigLong`: the diff rewrites unrelated Long paragraphs (--print, list key, open map, preserved keys) to fit the byte cap. Settle whether each rewrite keeps its original meaning and whether any of it is outside the text-only change the decision record asks for."
  - "internal/configcli/configcli.go: does the new `lym config shuttle --layer default --set deny_python=false` line name the switch's effect on both the hook and the notice, as the record asks for `lym config shuttle` and the template? Check the command is correct for the layer semantics and prime-only write."
  - "internal/shuttleengine/template.yaml `deny_python` comment and config.go `DenyPython` doc: do they name the effect on both hook and notice, and is the template comment still valid for the golden/config tests that read it?"
  - "internal/shuttleengine/claudeengine/testdata/settings-golden.json: check the golden diff is only the new notice wording and matches what `buildSettings` produces for the python row, with no unrelated drift."
  - "Test economy in internal/shellguard/rows_test.go and internal/configcli/configcli_test.go: the new python assertions are on static prose; check them against the rubric's test-economy bullets and against existing coverage by the golden file."
  - "Scope against the record: the record says text change only with no new switch; confirm the diff adds no behavior beyond wording, and check the `_lym/landing/summary.md` claims against the diff."
---

Seed focus from the decision record's `## Goal` and the committed diff over the review range.
