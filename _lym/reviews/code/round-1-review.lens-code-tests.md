```yaml
lens: code-tests
outcome: reported
```

**Per behaviour, the test that pins it**

1. **Python steer, Codex steer and notice state the reason, the operator-or-parent way forward and the no-work-round rule.**
   - Pinned by `TestCheckTexts` in `internal/shellguard/rows_test.go:48-56`. It checks the plain steer, `SteerFor(engineCodex)` and the notice for five phrases.
   - The notice that reaches a session is also pinned byte for byte by `settings-golden.json`, read in `internal/shuttleengine/claudeengine/settings_test.go:988`. The golden covers the notice wording only, not the steers.
   - Every test here asserts prose, so the behaviour is covered; the test-economy bullet is the issue (finding F1).
2. **The new text has no quote or backslash, including the `deny_python: false` form.**
   - Pinned by `checkTexts` at init, plus the shipped-rows-pass assertion in `TestCheckTexts`. A violation would fail at package init and in that test.
3. **The `otherwise` prefix on `pythonRoutes`.**
   - Only the prose assertions and the golden pin it, and the golden covers the notice only. The steer's `otherwise` itself has no row, and it is static prose, so it is exempt.
4. **`lym config` Long names the hub-wide command `lym config shuttle --layer default --set deny_python=false`.**
   - Pinned by the new `wants` entry in `TestConfigLong` (`internal/configcli/configcli_test.go:284`). It matches the literal command, so a wrong layer or key would fail it.
5. **The Long rewrites (`--print`, list key, open map, preserved keys).**
   - No test pins them, and none is needed: they are prose.
   - The 2000-byte cap that motivated the second commit has no test either. A grep of `configcli` and `shellguard` tests found no check of the Long length.
   - Meaning checks on the rewrites belong to the handler's review, not this lens.
6. **`template.yaml` comment and the `config.go` `DenyPython` doc.**
   - Static prose, so exempt. Existing config and template tests only load the file, and this change does not alter that.
7. **`doc.go` additions in `shellguard`, `configcli` and `claudeengine`.**
   - Prose only, exempt.

**Findings**

- **F1, LOW, `consistency`.** Test economy: the new python loop in `TestCheckTexts` (`internal/shellguard/rows_test.go:48-56`) asserts static prose.
  - The five phrases pin wording of the steer and notice text. The rubric lists "a new assertion on the static prose of a stencil, spec, doc or other text" as a finding.
  - The notice wording is already pinned by `settings-golden.json` (`settings_test.go:988`).
  - Fix: remove the loop, or keep only the part that is behaviour, which is that the Codex steer keeps the shared refusal, and drop the phrase list.
- **F2, LOW, `consistency`.** The new `TestConfigLong` entry (`internal/configcli/configcli_test.go:284`) asserts the literal help-prose command line.
  - It is a literal-prose assertion on help text. It does pin a command that must stay correct, so it is borderline, but it falls under the same test-economy bullet.
  - Fix: keep it only if the handler judges the command a behaviour contract, otherwise remove it.

No behaviour in the diff is left without a pinning test.

**Tool calls**
1. Bash grep for `2000` in the `configcli` and `shellguard` tests and for the golden test in `claudeengine`. My context did not hold the existence of a Long-length test or the golden test's location.
