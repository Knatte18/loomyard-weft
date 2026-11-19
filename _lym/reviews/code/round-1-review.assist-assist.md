---
verdict: APPROVED
findings:
  - id: F1
    severity: "MEDIUM"
    class: "design"
    location: "internal/shellguard/rows.go:148"
    summary: "Escaped double quotes hide real removals and turn quoted argument text into commands."
    fix: "Update the double-quoted script scan to distinguish JSON escapes, outer shell quotes and escaped quotes passed to the inner shell. Add existing-table rows for allowed quoted separators and a denied computed removal after an escaped quoted argument, covering both script quote forms. Refresh the settings fixture."
    origin: "lens:code-correctness"
  - id: F2
    severity: "MEDIUM"
    class: "design"
    location: "internal/shellguard/rows.go:152"
    summary: "Script scanning omits refusals on a heredoc opening line."
    fix: "Extend shellScriptReachPattern and redirectWritePattern with script-aware heredoc opening-line traversal equivalent to topLevelReachPattern. Keep heredoc bodies excluded. Add rows for a computed removal piped from a heredoc and a source redirect after its operator. Refresh the settings fixture."
    origin: "lens:code-correctness"
  - id: F3
    severity: "MEDIUM"
    class: "design"
    location: "internal/shellguard/rows.go:412"
    summary: "Quoted protected-variable expansions inside single-quoted scripts evade the raw-Go row."
    fix: "Extend the script-aware environment-variable branch to recognize expansions inside inner double quotes while preserving literal-name allowances. Add a regression row for a protected-variable expansion inside a single-quoted shell script. Refresh the settings fixture."
    origin: "lens:code-correctness"
  - id: F4
    severity: "MEDIUM"
    class: "design"
    location: "internal/shellguard/rows.go:408"
    summary: "Raw-Go matching treats outer shell arguments as arguments of the inner Go command."
    fix: "Bound the raw-Go argument scan to the selected script's closing delimiter. Add an allowed row for sh -c 'go build ./cmd/lym' all and retain the denied inner all case. Refresh the settings fixture."
    origin: "lens:code-correctness"
  - id: F5
    severity: "MEDIUM"
    class: "scope"
    location: "internal/shellguard/rows.go:134"
    summary: "The shared keyword prefix adds top-level refusals outside the requested shell-script change."
    fix: "Keep the existing top-level command prefix and apply new command-opening keywords only inside script traversal. Adjust the top-level keyword test rows and documentation to preserve baseline behavior. Refresh the settings fixture."
    origin: "lens:code-scope"
  - id: F6
    severity: "LOW"
    class: "consistency"
    location: "internal/shuttleengine/claudeengine/doc.go:681"
    summary: "Claude's per-row bypass lists still claim ordinary shell -c scripts evade the guards."
    fix: "Update the bypass lists at doc.go lines 706, 723, 733, 747 and 761 to match script-aware behavior. Reconcile the do/then claim at line 734 with the scope fix. Retain only remaining limitations such as nested scripts."
    origin: "lens:code-affected-set"
---

### [MEDIUM] Escaped double quotes hide real removals and turn quoted argument text into commands.

**Location:** internal/shellguard/rows.go:148
**Issue:** The generated computed-rm hook allows `bash -c "echo \"hello\"; rm \$d"`.
The inner shell removes a computed path after a quoted argument, leaving the Goal's dialog risk.
Conversely, `bash -c "echo \"x; rm \$d\""` is denied although it only echoes text.
The scanner confuses escaped inner quote delimiters with the outer script delimiter.
The tests lens's missing negative quoted-separator coverage is consolidated here.

### [MEDIUM] Script scanning omits refusals on a heredoc opening line.

**Location:** internal/shellguard/rows.go:152
**Issue:** Top-level reach retains an explicit heredoc opening-line branch, but script reach and redirects omit it.
The generated hooks deny `cat <<EOF | rm $d\nx\nEOF` and `cat <<EOF > f.go\nx\nEOF`.
Wrapping either command in `sh -c` with a single-quoted script makes it allowed.
These are opening-line operations, not ignored heredoc body text.

### [MEDIUM] Quoted protected-variable expansions inside single-quoted scripts evade the raw-Go row.

**Location:** internal/shellguard/rows.go:412
**Issue:** The generated raw-Go hook denies `echo "$LYM_GATE_SLOT"` but allows `bash -c 'echo "$LYM_GATE_SLOT"'`.
The new branch only recognizes names outside inner quotes.
The top-level quoted-expansion branch cannot enter the outer single-quoted script.
The Goal requires the same existing refusal inside the script.

### [MEDIUM] Raw-Go matching treats outer shell arguments as arguments of the inner Go command.

**Location:** internal/shellguard/rows.go:408
**Issue:** The generated raw-Go hook denies `sh -c 'go build ./cmd/lym' all`.
The shell assigns `all` to its unused `$0`; Go receives only the package argument.
The same script without the outer argument is allowed.
The reused argument suffix scans beyond the closing script quote.

### [MEDIUM] The shared keyword prefix adds top-level refusals outside the requested shell-script change.

**Location:** internal/shellguard/rows.go:134
**Issue:** The Goal asks to inspect shell -c scripts using existing refusals.
The unconditional shared keyword prefix also changes top-level loops without a shell -c.
The new `sed_in_place_after_do` and `rm_after_do` rows require that expansion.
The landing summary acknowledges it, but the decision record has no amendment and the commits have no departure trailer.
Keyword handling inside the incident's loop is within scope; changing the outer command grammar is separable.

### [LOW] Claude's per-row bypass lists still claim ordinary shell -c scripts evade the guards.

**Location:** internal/shuttleengine/claudeengine/doc.go:681
**Issue:** The changed overview says shell -c scripts are inspected.
The per-row descriptions at lines 706, 723, 733, 747 and 761 still list ordinary bash -c as a bypass.
Line 734 also describes the old do/then behavior.
The landing summary's claim that package documentation no longer lists bash -c bypasses is inaccurate.

## Review evidence

The review range is 7346d6a8c8b38d0b26987969ac76b6b3843727c6..08ebbcf63aba037703b235036f3735ae5e3c85b4.
The board entry matches the decision record's Goal.
The correctness lens executed generated grep hooks over payloads without executing their command text.
The incident's exact loop shape is denied, and `echo do rm $x` remains allowed.
Empty scripts, literal removals, fd duplication and quoted redirect targets retain their expected allowances.

Decoded fixture comparison found only hook-command changes, with denial suffixes unchanged.
Both engine renderers consume the shared rows.
The python expectation flip implements the requested behavior and is not a loosened check.
The added table cases use independent literal expectations and introduce no sleeps.

The handler's initial combined probe was refused by the session guard and did not run.
A later fixture comparison used jq because Node was unavailable.
A harmless payload passed through the generated computed-rm hook without a denial.
No target files were changed.
