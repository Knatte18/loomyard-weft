```yaml
lens: code-correctness
outcome: reported
```

Task: review the code-correctness lens of the python guard diff.

**List (function or branch, case run, result)**

- `pythonRefusal`, steer, plain: forbidden-character set is in `rows.go`, but the text has no quote or backslash. `string(SwitchDenyPython)` is `deny_python`, so the `: false` form adds only a colon and a space. `checkTexts` at init passes. Correct.
- `pythonRefusal`, Codex steer: `pythonRoutes + codexEditToolsRule` carries the same refusal text, so it holds the reason and the way forward. `checkTexts` iterates `EngineTexts` as well. Correct.
- `noticePythonDeny` = `pythonRefusal + "."`: the notice text has the `;` and `:` that other shipped notices already carry, such as the AskUserQuestion notice and the "Gate rule:" text. No newline, no quote. Correct.
- `pythonRoutes` = refusal + `"; otherwise parse JSON..."`: reads as "...and never work round the guard; otherwise parse JSON with jq...; change a file by reading...". The grammar holds. The meaning is slightly off: "otherwise" attaches to the "never work round the guard" clause, not to "when the work needs Python". Cosmetic only; see F1.
- `buildConfigLong`: the key name comes from `shellguard.SwitchDenyPython`, so the line renders as `lym config shuttle --layer default --set deny_python=false`. `TestConfigLong` asserts that exact string, so it matches. The text still names `--set`, `--layer`, `$VISUAL` and the editor fallback. Correct.
- `buildConfigLong`, the rewritten `--print`, list-key, open-map and preserved-keys paragraphs: each keeps its original meaning. One loss: the old "A module may declare an open map" is now "An open map's keys are the repo's own", with the same sense. The reconcile sentence is reordered but unchanged in what it commands. Correct.
- `buildConfigLong`, import of `shellguard` into `configcli`: no cycle is visible from the diff. `shellguard` imports only stdlib, per `PATTERN-shell-guard-table`. Correct.
- `template.yaml`, `deny_python` comment: still one `key: value  # comment` line. The comment holds `: ` only after the `#`, so it does not change parsing. `config.go` `DenyPython` doc change is comment-only. Correct.
- `settings-golden.json`: only the notice and settings strings that carry the python row changed, per the diff lines I saw. The notice text matches `noticePythonDeny` with `deny_python` interpolated. I saw no unrelated drift.
- Behavior: no matcher, switch or condition changed. `pythonCommandPattern` is untouched.

**Findings**

F1 — NIT, consistency: `internal/shellguard/rows.go:82`. "otherwise" after the refusal's "never work round the guard" binds to the wrong clause, since the parse routes apply to work that does not need Python. Fix: say "for work that does not need Python itself, parse JSON with jq...", or drop "otherwise".

No other defects found.

**Tool calls**
1. Read `rows.go` 525-565 for `checkTexts` and the init check: the manifest showed only the call sites.
2. Grep of `internal/configcli` and `internal/cliwire` for the 2000-byte cap and the `--layer` semantics: to check the Long text and the layer wording. It found no cap constant in these packages; I did not locate the cap's test.
