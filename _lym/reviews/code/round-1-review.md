---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/shellguard/rows_test.go:48
    summary: new python loop in TestCheckTexts asserts five phrases of static prose that the golden file already pins for the notice
    fix: In TestCheckTexts, replace the loop over five phrases with one assertion that the plain steer, the Codex steer and the notice each carry the shared refusal text (strings.Contains of pythonRefusal is not reachable from the external package, so compare the Codex steer against the plain steer's shared prefix up to the edit rule), or delete the loop; the notice wording stays pinned by settings-golden.json.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/configcli/configcli_test.go:284
    summary: new TestConfigLong entry asserts a literal command line in help prose
    fix: Remove the "lym config shuttle --layer default --set deny_python=false" entry from the wants list in TestConfigLong and restore the doc comment's earlier wording, since the command is static help prose.
  - id: F3
    severity: NIT
    class: consistency
    origin: lens:code-correctness
    location: internal/shellguard/rows.go:82
    summary: otherwise after the refusal's final clause reads as attached to never work round the guard instead of to work that does not need Python
    fix: In pythonRoutes, replace "; otherwise parse JSON with jq and text with grep or awk; " with "; for work that does not need Python itself, parse JSON with jq and text with grep or awk; ", and regenerate settings-golden.json so the notice and steer fixtures match where they carry the steer.
---

### [LOW] Python loop in TestCheckTexts asserts static prose

**Location:** internal/shellguard/rows_test.go:48
**Issue:** The new loop pins five phrases of the python steers and notice. The rubric's test-economy bullet names a new assertion on static prose as a finding. The notice wording is already pinned byte for byte by `settings-golden.json` (read in `internal/shuttleengine/claudeengine/settings_test.go:988`). The Codex steer is the one text the golden does not cover, so the only part worth keeping is that the Codex steer carries the shared refusal. The behaviour itself, the `checkTexts` init check against quotes and backslashes, stays pinned.

### [LOW] TestConfigLong asserts a literal help-prose command

**Location:** internal/configcli/configcli_test.go:284
**Issue:** The new `wants` entry matches a whole command line in help text. It is static prose, so the rubric's test-economy bullet applies. The command's correctness is a prose claim and no behaviour fails if it drifts.

### [NIT] otherwise binds to the wrong clause

**Location:** internal/shellguard/rows.go:82
**Issue:** `pythonRefusal` ends with "and never work round the guard", and `pythonRoutes` then adds "; otherwise parse JSON...". Read in sequence, "otherwise" answers "never work round the guard" rather than "when the work itself needs Python". The sentence still parses and the meaning is recoverable. The notice does not carry the clause, so only the steers read this way.

## Rejected

- Unrelated `Long` paragraph rewrites in `internal/configcli/configcli.go` (--print, list key, open map, preserved keys): each keeps its original meaning and the commit `dce424a4` states its purpose, the 2000-byte cap enforced by `maxLongBytes` in `cmd/lym/clitree_test.go`; a cap the verify gate decides, and the rewrite is text-only, so no scope finding.
- The `shellguard` import into `configcli`: `PATTERN-leaf-packages` limits what `shellguard` imports, not who imports it, and `claudeengine` and `codexengine` already import it.
- Missing reason or way forward in the Codex steer: `codexSteerPythonDeny` is built from `pythonRoutes`, so it carries both.
- Quote or backslash risk from the `: false` form: the init check passes, as `forbiddenSteerChars` holds only `'`, `"` and `\`.
- Golden drift: the diff changes only the settings and notice strings that carry the python row.
- `docs/research/tokens/startup-context-2026-11-05/classify.jq` still matching the old wording: a dated measurement, and the change description says it stays as it ran.
- Behaviour beyond wording: `pythonCommandPattern`, the switch and the matchers are untouched; no new switch is added, as the record asks.
