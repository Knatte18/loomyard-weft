---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/boardengine/board.go:225
    summary: The entry asks for a test pinning that each rendered way-forward command parses, but only the front-task refusal reaches the remove verb's parser; RemoveCommand, the one renderer all sites share, has no test of its own.
    fix: Add a table-driven test in internal/boardengine (external test package) that takes RemoveCommand(slug) for a few slugs, extracts the single-quoted payload, and feeds it to the remove verb's lookup parsing (the code behind `lym board remove`, lookupPayload/resolveLookup, via an export_test.go seam if needed), asserting it resolves to the same slug. The adoptcli and fronttask string assertions then stay as they are.
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: scope
    location: internal/adoptcli/green.go:203
    summary: The changed fallback line for an error that is not a RemoveError has no covering test row.
    fix: In internal/adoptcli/green_test.go, add a row to TestGreenEnd where the fake's removal returns a plain error that is not a *fronttask.RemoveError, and assert the line names "lym board remove '{\"slug\":\"adopt-fix-alpha-t1\"}'". If the branch cannot be reached because fronttask.Remove always returns a RemoveError, delete the fallback branch instead.
    origin: lens:code-tests
---

### [MEDIUM] No test pins that every rendered way-forward command parses

**Location:** internal/boardengine/board.go:225; internal/adoptcli/fixloop_test.go:257; internal/adoptcli/green_test.go:390; internal/fronttask/fronttask_test.go:152
**Issue:** The decision record's Expected says "a test pins that each way-forward command these packages render parses".
`stepFrontTask` in `internal/boardcli/verbs_test.go` runs the real remove verb with the payload the front-task refusal names, so it covers that one site.
The adoptcli and fronttask tests compare strings only.
All sites render through `RemoveCommand`, so one test of that function against the verb's parser covers them; none exists.

### [LOW] Fallback removal line in green.go has no covering row

**Location:** internal/adoptcli/green.go:203
**Issue:** The `removeFixTask` branch for an error that is not a `RemoveError` changed text, and no row in `green_test.go` reaches it.
The existing row takes the `errors.As` branch through `HandRemoval`.

## Rejected

- Single quote in a slug breaks `RemoveCommand`'s shell quoting (code-correctness, LOW; code-tests F-T2 in part): slugs pass `agentname.ValidateSlug`, which accepts dashed names only, so `'` cannot occur in a slug a way forward names.
- Discarded `json.Marshal` error: `json.Marshal` of a `map[string]string` cannot fail.
- New exported helper not licensed by the entry: the entry asks every way forward to name an accepted command, and one renderer keeps that form in one place; no layering rule is crossed, and `fronttask` already imported `boardengine`.
- Remaining bare-slug text elsewhere in the tree: the whole-tree grep found only text with no slug argument and the verb's own help, which already shows the JSON form.
- Separate escaping-row test for `RemoveCommand` (code-tests F-T2): folded into F1; slugs are restricted, so escaping rows would pin nothing reachable.
