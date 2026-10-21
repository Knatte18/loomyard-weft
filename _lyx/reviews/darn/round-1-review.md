---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: internal/shuttleengine/claudeengine/doc.go:196-218
    summary: The claude engine module doc still describes the old text-anywhere guard and has no paragraph for the new redirect-write row.
    fix: In internal/shuttleengine/claudeengine/doc.go, rewrite the python, raw go, in-place edit and computed-rm paragraphs to the quote-aware behaviour as the revised comments in internal/shellguard/rows.go state it (quoted text, heredoc bodies and other payload fields no longer trip the rows; the raw go variable names are refused when set or read outside quotes or expanded in a double-quoted string, and the prebuilt-binary variable is named; the remaining false denies are only what the scan misreads). Add a paragraph for the redirect-write row, installed for every run including the orch with no config key, naming what it refuses, what it allows (.scratch/ by relative path, /tmp/, /dev/, fd duplication, variable-led targets) and its guardrail limits.
    origin: lens:code-pattern-fit
  - id: F2
    severity: LOW
    class: design
    location: internal/shellguard/rows.go:35,52
    summary: A command substitution inside a double-quoted string is never in command position, so out="$(python3 x.py)" or echo "$(go test ./...)" now passes every command-shaped row; the old pattern denied it and no comment lists it as a pass-through.
    fix: Either add a separator alternative to commandPositionPattern (and the raw go variable alternatives) that reaches a $( or backtick inside a double-quoted body, with a deny row for it in rows_integration_test.go, or state under each affected pattern's let-through comment and in internal/shellguard/doc.go that a substitution inside a double-quoted string passes.
    origin: lens:code-correctness
  - id: F3
    severity: LOW
    class: design
    location: internal/shellguard/rows.go:35,43,52
    summary: A < before a quote or backslash (sort <"f" | python3 x) ends the unit run, and <<< is read as a heredoc operator with no body, so commands after either are unreachable; the common write-then-run heredoc shape also leaves later lines unguarded, which doc.go does not state.
    fix: Read <<< as a here-string unit, not a heredoc operator, and let a lone < be followed by a quote or escape in shellUnitPattern and shellLineUnitPattern; or list these under the pass-through comments. State in internal/shellguard/doc.go that nothing after a heredoc operator's line is read.
    origin: lens:code-correctness
  - id: F4
    severity: LOW
    class: design
    location: internal/shellguard/rows_integration_test.go:85-274
    summary: Test gaps in the new behaviour - no row puts a command word in another payload field such as description, the separator forms (subshell, backtick, $(, <( , >() are pinned on raw-go only, and redirect-write has no single-quoted target or /dev lookalike row.
    fix: In payloadDenied, let a row carry a description field after the command, and add a row per command-shaped row showing a command word there is allowed. Add rows to the existing table - a backtick, $( and subshell deny on python, a single-quoted redirect target deny, and a /devx lookalike deny on redirect-write. No new test function.
    origin: lens:code-tests
---

### [MEDIUM] Claude engine module doc left stale

**Location:** internal/shuttleengine/claudeengine/doc.go:196-218
**Issue:** CLAUDE.md requires the module doc to land with an observable behaviour change. The doc's row paragraphs still say quoted arguments, grep alternations and heredoc bodies are falsely denied, name only two raw-go variables, and omit the new always-installed row.

### [LOW] Substitution inside double quotes passes

**Location:** internal/shellguard/rows.go:35,52
**Issue:** The shell runs `$(...)` inside a double-quoted string, but the scan treats the whole string as one unit. This is a regression from the old pattern and is not listed as a pass-through.

### [LOW] Unreached commands after `<` forms and heredocs

**Location:** internal/shellguard/rows.go:35,43,52
**Issue:** `<` before a quote, `<<<`, and any command on a line after a heredoc operator's line fall outside the scan, so rows do not see them. The package doc does not say so.

### [LOW] Coverage gaps in the new rows

**Location:** internal/shellguard/rows_integration_test.go:85-274
**Issue:** The `description`-field case the summary and comments claim is untested, as are most separator forms outside raw-go and two redirect-write branches.

## Rejected

- Fork finding that the redirect-write steer cites an edit directive that does not name redirects: the directive says to change files only with Edit or Write, which covers redirect writes, and the in-place-edit steer cites it the same way.
- Fork nit on the duplicated computed-rm rows in `hookcmd_integration_test.go`: they pin the multi-pattern rendering, which no other test covers.
- Fork finding that the heredoc write-then-run hole is a design flaw: the board entry asks only for the quote-aware match and redirect refusal, and the limit is documented in the pattern comments; the doc.go gap is folded into F3.
