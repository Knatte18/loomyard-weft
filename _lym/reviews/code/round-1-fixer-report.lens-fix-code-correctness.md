```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix-code-correctness lens over the three fixes.

No findings.

- **F1, `rows_test.go`:** The `python` row's Codex steer is `pythonRoutes + codexEditToolsRule`, and `pythonRoutes` opens with `pythonRefusal`. `strings.HasPrefix(row.SteerFor(engineCodex), pythonRefusal)` therefore passes today. It fails if the Codex steer drops or rewords the refusal, because the Codex text is built from `pythonRoutes`. `pythonRefusal` and `engineCodex` are both package-level in `package shellguard`, which the test file uses, so both resolve. The deleted loop also covered the plain steer and the notice, and settings-golden.json pins both byte for byte, so nothing is lost.
- **F3, `rows.go` and golden:** The new text has no `'`, `"` or `\`, so `checkTexts` and the init check still pass. The notice never carries `pythonRoutes`, which matches the golden: the notice lines are untouched and only the eight `denyPython=true` settings strings changed. The gate shows `TestBuildSettings_MatchesGolden` green, and the golden is only the Claude engine's settings file. In the earlier grep the old phrase appeared only in `rows.go` and that golden, so no other fixture is left stale.
- **F2, `configcli_test.go`:** The removed `wants` entry has no other reader, and the doc comment is back to its earlier wording.

Gate result (reported by the parent, not re-run): `lym gate test` over shellguard, configcli and claudeengine passed 914 and failed 0.

No further tool calls were made; the context already held everything needed.
