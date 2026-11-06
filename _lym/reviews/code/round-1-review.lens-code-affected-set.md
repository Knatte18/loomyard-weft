```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. Everything the python-guard wording change forces outside the touched lines.

**Per change, the set I found and whether it is covered:**

1. **`pythonRefusal`, `pythonRoutes` and `noticePythonDeny` in `rows.go`.**
   - The text reaches the claude steer and notice and the codex steer, and `buildSettings`/`buildDenyNotice`, which read the row, pick it up unchanged.
   - Covered: `settings-golden.json` carries the new notice text, with all the `settings` and `notice` pairs shifting together.
   - Covered: `TestCheckTexts` pins the steer, the codex steer and the notice.
   - Covered: `cmd/lym/norecoverydeny_test.go` reads `shellguard.Rows`, not the text, so it is unaffected.
   - Covered: the init check for quotes and backslashes still holds. The new text has none, and `string(SwitchDenyPython) + ": false"` adds none.
   - Search for other literal copies of the old wording: only the golden, both `doc.go` files and `rows.go` hold it. `docs/research/tokens/.../classify.jq` (the summary's note) is outside my repo-wide grep because it matched `deny_python` rather than the old wording. It is a dated measurement the summary says to leave alone, which is acceptable.
   - Covered: `shellguard/doc.go` and `claudeengine/doc.go` both got a line for the new text.
2. **`Long` of `lym config`.**
   - The new `shellguard` import: `PATTERN-leaf-packages` constrains what `shellguard` imports, not who imports it. Other non-test packages already import it (`claudeengine/settings.go`, `codexengine/guards.go`), so this is allowed.
   - The 2000-byte cap lives in `cmd/lym/clitree_test.go:167,274`, `longProblems` over every command. The second commit's rewrites exist to stay under it. I did not measure the length, so the cap result is unconfirmed.
   - `configcli_test.go` `TestConfigLong` pins the new command line, and `configcli/doc.go` was updated.
3. **`template.yaml`, `config.go` `DenyPython` and `template.go`.**
   - The template comment and the `DenyPython` doc both name the effect on hook and notice.
   - `shuttleengine/doc.go:57` and `template.go:15` mention `deny_python` without describing its effect. Nothing there is made wrong.
4. **`languageconfig/doc.go:25`.** It says a Python rename tool needs `deny_python` off. That stays true, and the wording does not conflict.
5. **PATTERN rules.**
   - `PATTERN-refusal-way-forward` binds only webster, shed and loom. The new steer offers no denied step, and the way forward is the operator's config switch.
   - No `PATTERN.md` or `pattern/` change is declared, and none is needed.
   - No commit carries a `Pattern-Departure:` trailer, and none is needed.
6. **Commit messages.** Both commits are consistent with the diff. The second is a cap fix.

**Findings:** none.

One note outside my lens: the second commit rewrites unrelated `Long` paragraphs (--print, list key, open map, preserved keys). The focus list asks for this to be judged, so the handler should settle whether it stays in scope.

**Tool calls (3):**
1. Grep for the old wording and `deny_python` across docs and the shuttle packages, plus the repo-wide old-text search and the `2000` / `PATTERN` greps. My context lacked other readers of the old text.
2. Read the `PATTERN-leaf-packages` shellguard section, the shellguard importers and the `maxLongBytes` cap. My context lacked who may import `shellguard` and where the cap is enforced.
3. Grep `norecoverydeny_test.go`, the `Long` checks in `clitree_test.go` and the head of `PATTERN-refusal-way-forward`. My context lacked whether the new text trips the no-denied-recovery scan.
