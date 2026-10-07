---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:223
    summary: The PATTERN-fabric-vocabulary Constraint omits that the scan is a case-insensitive substring match and that only three spellings are exempt, so the plan writer cannot write the fixtures D8 and D13 to D15 need
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:10
    summary: The Goal says a card has a verify section and D13 says the finding names the card, but the verify section is plan-level and fenced blocks outside cards are not placed in or out of scope
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:174
    summary: D13 names the shared unit as three patterns, but the quote blanking and heredoc stripping that give outside quotes its meaning, and the separate path and spelling entry points with matched text, are not stated
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:151
    summary: D11 never names which runs contribute findings to a profile line and leaves the denominator undefined when none does
---

### [MEDIUM] The vocabulary Constraint does not give the plan writer the scan's real shape

**Location:** `_lyx/discussion/decision-record.md`, Constraints, the `PATTERN-fabric-vocabulary` entry.
**Issue:**
The entry says the bare tokens stay on the fabric side and that the enforcing test "scans `planparser`, every test file, `contracts/` and the docs".
It does not say what `bareVocabularyToken` in `internal/lyxcwd/enforcement_test.go` does, which decides what the plan writer can write:

- The match is a case-insensitive substring over identifiers, string literals and comments, so any literal carrying `weft` or `warp` fails, including a `-weft` flag name or a path ending in `-weft`.
- `vocabExempt` strips a closed list of whole tokens first.
  Of the spellings D13 refuses, only `lyx weft sync`, `lyx warp checkout` and `lyx.exe weft push` are on it.
  A fixture for the `lyx weft` or `lyx warp` spelling written any other way (`lyx weft push`, `lyx warp sync`) fails the scan.
- Test files are scanned wherever they sit, `tools/tokencount`'s included, while non-test `.go` under `tools/` is not.
  D8's acceptance criterion ("accepts `-weft` alone for `-calibrate`, rejects `-history`") therefore cannot be tested with the flag spelled in a test file; the name has to come from a non-test constant.

The existing audit tests (`internal/websterengine/audit_test.go`) pass only because they use exactly the three exempt spellings and `fabricengine.WeftWorktree`.
The record's last sentence of the entry ("build weft-named inputs with the fabric side's exported helpers") covers paths but not spellings or the `-weft` flag.
Without this, the plan's test cards for D13 to D15 and D8 fail `TestEnforcement_FabricVocabulary` and bounce.
**Fix:**
State in the entry that the scan is a case-insensitive substring match with a closed exempt list, name the three exempt spellings as the only `lyx weft` / `lyx warp` spellings a fixture outside the fabric side may carry (otherwise `lyx fabric …` or a helper-built string), and say that tokencount's tests take the flag name from a non-test constant.

### [LOW] Scope of the scanned text does not match what the Goal and the finding say

**Location:** `_lyx/discussion/decision-record.md`, Goal (line 10) and D13 (lines 168 to 172).
**Issue:**
The Goal refuses "a card whose `**Verify:**` value, verify section or fenced code block …", but the `## verify:` section is plan-level, in `00-overview.md`, not a card's.
D13 says "the plan's `## verify:` section" correctly, then says the finding "names the card", which has no meaning for a hit in the plan-level section.
D13 also scans "fenced code blocks" without saying whether that means fenced blocks in cards only or also in the overview's `## Shared Decisions` and `## Rename mechanic`, where a script may sit.
The scope of the sibling-path rule and the Bound agree with each other and with the sibling-name rule: a relative path such as `../<slug>-weft` and an absolute path both carry the name and are refused, and a path without the name is not.
**Fix:**
Reword the Goal so the verify section is the plan's, state in D13 which fenced blocks are scanned, and say how the finding labels a hit outside any card.

### [LOW] D13's shared rule is described as patterns; the shared unit is larger and the seam needs two scopes

**Location:** `_lyx/discussion/decision-record.md`, D13 (lines 174 to 177) and the matching Constraint.
**Issue:**
D13 says the shared rule is "the three location-free patterns".
`RefScanner.Matches` also strips heredoc bodies and blanks quoted spans before the bare-word and spelling patterns run, and "a bare word … outside quotes" in D13 only has a meaning through that blanking.
A writer who shares only the regexes cannot implement "outside quotes", and a fenced block holding a heredoc is judged differently by the plan check and the audit.
D13 also needs the path rule and the spelling rule applied to different spans and the finding to carry the matched text, while the `RefMatcher` it points to ("as `websterengine` receives its matcher") is one `Matches(cmd) bool`.
**Fix:**
Say the shared unit is the matching including heredoc stripping and quote blanking, and that the seam exposes the path rule and the spelling rule separately and returns the matched text.
No signature is asked for.

### [LOW] D11 leaves the profile line's findings denominator open at the edges

**Location:** `_lyx/discussion/decision-record.md`, D11 (lines 149 to 152) and the Acceptance criterion on the per-profile summary.
**Issue:**
The denominator is defined: "the findings over the cards of only the runs that contribute findings", with the count of those runs stated, and multi-profile and restarted runs enter neither side.
Two things are left to the plan writer.
The set "runs that contribute findings" is never named (a run with one profile, not restarted, with readable review files).
A profile whose runs all fail that test has no denominator, and the record does not say the figure is then marked not available instead of printed as zero, which "never a silent zero" requires.
**Fix:**
Name the contributing set in D11 and state that a profile line with no contributing run shows the figure as not available.

## Focus departures

None.
Verdicts on the four directive questions:

1. Constraints do not state the vocabulary rule well enough: F1.
2. The Goal's guarantee sentence and D13's Bound state the same scope for the sibling path (anywhere in the scanned text, inline spans included) and for the spelling (Verify value, verify section, fenced blocks only), and the Bound's wording on absolute paths and `..` escapes agrees with the sibling-name rule; only the "card's verify section" wording is off: F2.
3. Not a defect.
   The exact-path alternative in `NewRefScanner` has no name-end anchor, so the audit matches the current worktree's weft path as a prefix (`<path>-bare`, say), and D13 says that alternative stays in the audit and is outside the shared rule, so "the audit's matches stay unchanged" is true of it.
   The only gap in the description is the preprocessing, F3.
4. The denominator is defined; the open edges are F4.
