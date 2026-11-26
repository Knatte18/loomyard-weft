---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/shellguard/doc.go:40
    summary: shellguard's package doc now documents the Claude engine's settings golden and its test command, which breaks PATTERN-code-conventions' rule that a doc.go documents only its own package
    fix: "Delete the two-line paragraph in internal/shellguard/doc.go that opens \"A row change rewrites the Claude engine's settings golden\" and gives the lym gate test command, together with the blank comment line that separates it. The command already sits in the updateSettingsGolden doc comment in internal/shuttleengine/claudeengine/settings_test.go, which is its own package's place for it."
  - id: F2
    severity: BLOCKING
    class: consistency
    origin: lens:code-affected-set
    location: internal/shuttleengine/claudeengine/settings_test.go:1153
    summary: the golden path testdata/settings-golden.json is written twice, as a literal in the -update flag's usage string and again in the goldenPath join, which breaks PATTERN-code-conventions' one-named-constant rule
    fix: "In internal/shuttleengine/claudeengine/settings_test.go, declare the golden path once at package level, e.g. var settingsGoldenPath = filepath.Join(\"testdata\", \"settings-golden.json\"), declared before updateSettingsGolden. Build the flag's usage string from it (\"rewrite \" + settingsGoldenPath + \" from the rendered settings and deny notices\"), and replace the local goldenPath := filepath.Join(...) in TestBuildSettings_MatchesGolden with that variable."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/shellguard/rows.go:428
    summary: the unquotedWordOtherThan doc and the package doc promise that the scan fails closed, but the helper returns only the word pattern; fail-closed comes from the caller's end alternation, which must hold every character the word unit excludes, and the docs name that end only as a separator, a quote or the payload's end, so an author following them can write a fail-open row
    fix: "In internal/shellguard/rows.go, give the end a single source: add a function beside unquotedWordOtherThan, e.g. unquotedWordRunEnd(), returning `([;&|()<>\"'\\\\`]|$)` (the searchEnd value now built inline in sessionBusCommandPattern), built from the same excluded-character string the word unit uses, so the two cannot drift. Use it in sessionBusCommandPattern in place of searchEnd. Rewrite the doc sentences of unquotedWordOtherThan to say that the run is fail-closed only when it is ended by that end pattern, naming it, and drop the claim that the helper itself matches at a stop. In internal/shellguard/doc.go, change \"matches at a separator, a quote or the payload's end\" to say the scan is ended by that same end pattern, which covers every character a word cannot hold."
  - id: F4
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md#Merge danger
    summary: the Merge danger blast radius names only shellguard, but the diff also changes the claudeengine test package's TestBuildSettings_MatchesGolden and adds a package-level -update flag there
    fix: "In _lym/landing/summary.md, under ## Merge danger, change the blast radius to name both shellguard and the claudeengine settings golden test (internal/shuttleengine/claudeengine)."
---

### [BLOCKING] shellguard doc.go documents another package's golden and command

**Location:** `internal/shellguard/doc.go:40`

**Issue:** The new paragraph states that a row change rewrites the Claude engine's settings golden and gives `lym gate test ./internal/shuttleengine/claudeengine -- -run TestBuildSettings_MatchesGolden -update`.
`pattern/PATTERN-code-conventions.md`, under "Go documentation", says a package's `doc.go` documents the package, and that a rule other code must follow is a PATTERN entry, never a `doc.go` paragraph.
This paragraph documents a test and command of `internal/shuttleengine/claudeengine`.
The constraint makes every violation BLOCKING.
The same command already sits in the `updateSettingsGolden` doc comment in `settings_test.go`.

### [BLOCKING] Golden path literal repeated in settings_test.go

**Location:** `internal/shuttleengine/claudeengine/settings_test.go:1153`

**Issue:** The `-update` flag's usage string holds `testdata/settings-golden.json` as a literal, and `TestBuildSettings_MatchesGolden` builds the same path again with `filepath.Join("testdata", "settings-golden.json")` at line 1168.
`PATTERN-code-conventions`, under "Named constants", requires a path that can go stale to be one named constant, declared once and never repeated as a literal.
A rename of the golden would leave the usage string stale with nothing to catch it.

### [MEDIUM] Fail-closed claim is broader than the helper gives

**Location:** `internal/shellguard/rows.go:428`

**Issue:** The new doc of `unquotedWordOtherThan` says "The run stops at anything it cannot read and matches there, so the guard fails closed".
The function returns only a word pattern (`rows.go:434-448`), and nothing in it matches at a stop.
The match at a stop comes from `searchEnd` (`rows.go:420`), which `sessionBusCommandPattern` builds inline.
It is fail-closed only because it holds every character the word unit at `rows.go:436` excludes, whitespace aside: `;&|()<>"'\` and the backtick, plus `$`.
Both the helper doc and `doc.go` name that end as "a separator, a quote or the payload's end".
An author who follows only those words, for another command, can write an end such as `([;&|"']|$)`.
That row lets a refused command through when its scan stops at `>`, a backtick or a JSON escape (`\n`, `\t`, `\"`).
The guidance the board entry's Part 2 asks for is then not a reliable fail-closed recipe.
The two character sets are kept in step by hand.
The rows the diff touches are correct for dbus-send; the defect is in the guidance and in the coupling.

### [LOW] Merge danger blast radius omits claudeengine

**Location:** `_lym/landing/summary.md#Merge danger`

**Issue:** The blast radius names `shellguard` alone.
The diff also restructures `TestBuildSettings_MatchesGolden` and adds a package-level `-update` flag to the claudeengine test binary.
The `two-way` door call holds: the diff changes only tests and comments, no record format, config meaning or outward push.

## Rejected

- Package doc and helper doc wording on the scan's word-level behaviour: checked against `unquotedWordOtherThan` and `sessionBusCommandPattern` for `--system` exactly, `--syst`, `--systemd`, `--system=1`, quoted, escaped and wrapped-shell inputs. "Only the flag ... stops the match" holds for the dbus-send form; the only overbroad claim is the fail-closed one in F3.
- `system_flag_of_a_later_dbus_send` as a redundant row: `system_flag_of_a_later_command` reaches the end through `&`; the new row is the only session-bus row that ends the scan at `;`. It also pins the exact second-command case the board entry warns about. The expectation `true` (denied) is the right direction.
- The `-update` mode lacks a test: it is test tooling, and the compare path checks the file it writes once that file is committed.
- `-update` through `lym gate test`: the words after `--` reach the tier's `{args}` (`internal/gatecli/cli.go:108`), the tier runs at the tree root (`tierrun.go:526`), and the `-run` filter blocks a carry skip. The documented command works.
- Compare path loosened: every check at the review base remains (missing label, settings difference, notice difference, extra label), and no `Loosened-Check:` trailer is needed.
- The summary's byte-for-byte claim: `json.MarshalIndent` over a map sorts keys, indents by two spaces and HTML-escapes, and the committed golden has that layout with 16 labels, a `settings` then `notice` key in each, and a trailing newline. Nothing in the diff contradicts the claim.
- No scope finding: every change traces to Part 1 or Part 2 of the board entry. The diff takes the AND-only branch the entry licenses.
