---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shellguard/rows_test.go:95
    summary: refusedRaw's guard for a refuse prefix with more words than the compile form has no test row; removing it would panic on a slice bound and no test would fail
    fix: In TestRows_RawGoSteerNamesOnlyUnrefusedCompileChecks add a row such as {"PrefixLongerThanForm", []string{"go build ./internal/x extra"}, " A compile check names its packages, as in go build ./internal/x or go vet ./internal/x, never ./... or all. Gate rule"}. If the gate config loader rejects a "/" in a prefix word, use a longer prefix of plain words, for example "go build x y z".
  - id: F2
    severity: NIT
    class: consistency
    origin: handler
    location: docs/overview.md (shuttle paragraph, raw go guardrail sentence)
    summary: the overview says the raw go deny steers the session to a package-scoped compile check, but the code offers each check only while no refuse_raw prefix refuses it, and offers none when all are refused
    fix: In the shuttle paragraph of docs/overview.md change "steering the session to a package-scoped compile check and `lym gate test`" to "steering the session to `lym gate test`, and to a package-scoped compile check the hub's `refuse_raw` does not refuse".
---

### [LOW] Refuse prefix longer than the compile form is untested

**Location:** `internal/shellguard/rows_test.go:95`, `internal/shellguard/rows.go:229`
**Issue:** `refusedRaw` guards `words[:len(prefixWords)]` with `len(prefixWords) <= len(words)`. Only a prefix with more words than `go build ./internal/x` reaches the false side, and no row in `TestRows_RawGoSteerNamesOnlyUnrefusedCompileChecks` has one. A change that dropped the guard would panic on the slice bound at `Rows` time and every existing row would still pass. The code is correct as written; this is a coverage gap in the one new behavior of the diff.

### [NIT] Overview states the compile-check steer unconditionally

**Location:** `docs/overview.md`, shuttle paragraph
**Issue:** The paragraph says the raw go guardrail steers the session "to a package-scoped compile check and `lym gate test`". `compileCheckSentence` in `internal/shellguard/rows.go` names each check only while no `refuse_raw` prefix refuses it and returns an empty sentence when every check is refused. `internal/shuttleengine/claudeengine/doc.go` and `internal/shellguard/doc.go` state the condition; the overview does not.

## Rejected

- Lens code-tests finding 2 (no `python3 --version` row): `python3_with_flag` in `rows_integration_test.go` already pins that a flag after `python3` in command position is denied, which is the same pattern path a version check takes; a second row would duplicate it. The wording itself is steer prose, exempt from a static-prose assertion.
- Focus item on the golden's churn: the diff of `settings-golden.json` pairs one `settings` and one `notice` line per rendered case, each changed only where the python or raw-go text changed; no unrelated key or ordering change was found.
- Focus items on scope: the diff makes the Idea's second bullet (steering wording and a guard answering with the allowed command). Reading the refused turns and the `CLAUDE.local.md` heddle trial are not code changes and are not in the diff. The plan-spec, `docs/overview.md` and `doc.go` edits state the new behavior or drop the whole-module compile spelling that the raw go guard refuses, so they fall under the Idea's wording and stencil route; no scope finding.
- Focus items on `compileCheckSentence`, a blank prefix, a longer prefix, flags or `./...` prefixes and an all-refused list: run by the code-correctness lens, each case correct and the steer well formed.
