---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/driftengine/diffmode.go:35
    summary: The MigrateConfig doc and doc.go say an argument inside a quoted shell string stays, but only one that ends the quoted string stays; a non-final one is removed.
    fix: In internal/driftengine/diffmode.go reword the MigrateConfig sentence and the skipFileArgument comment, and in internal/driftengine/doc.go the MigrateConfig sentence, from "an argument inside a quoted shell string such as sh -c '...' stays" to "an argument that ends a quoted shell string, as in sh -c 'docker run -e A={skip_file}', stays, since the quote that closes the string follows it". Add a row to the table in TestMigrateConfig_SkipFileArgument in internal/driftengine/diffmode_test.go, named AnArgumentInsideAQuotedShellStringIsRemoved, with command "sh -c 'docker run -e A={skip_file} img'" and want "sh -c 'docker run img'".
---

### [LOW] Quoted-shell-string wording overstates what the migration leaves

**Location:** internal/driftengine/diffmode.go:35, internal/driftengine/doc.go (MigrateConfig sentence)

**Issue:** `skipFileArgument` ends in `(\s|$)`, so `-e A={skip_file}` followed by whitespace matches wherever it stands.
For `sh -c 'docker run -e A={skip_file} img'` the argument is removed and the result `sh -c 'docker run img'` is balanced.
Only an argument directly followed by the closing quote, such as `sh -c 'docker run -e A={skip_file}'`, fails to match and stays.
The docs say any argument inside a quoted shell string stays, and the tests pin only the ending-quote case.
The behavior is sound (quotes stay balanced, no stray escaped spaces); the text and the tests do not describe it.

## Focus verdicts

- `driftBaselineEngine` is a named constant in `internal/loomcli/arm.go` beside `darnWriteEngine`, and `wiring.go` passes it to `loomrecipe.HasEngine`. No defect.
- Migration of `sh -c 'docker run -e A={skip_file}'` leaves the command whole and balanced.
  A backslash-continued command keeps each continuation: removing a line's argument leaves its break and indent, so the line before still continues, and no stray escaped space results.
  No defect beyond F1.
- Each of the three behaviors has a test that fails if it is reverted.
  The `openJudgeSession` ordering is pinned by the failed-snapshot test, which would find an open mark if the mark were written first.
  The hub-only refusal of `volatile_files` and `volatile_keys` is pinned by the two `TestPrepareRunSettings` rows.
  The expect pass recording of the volatile lists is pinned by the `DeepEqual` over a non-empty `Volatile`.
