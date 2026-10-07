---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:D13
    summary: D13 says quote blanking precedes all three shared patterns, but RefScanner runs the separator-prefixed path pattern on the heredoc-stripped text with quotes intact, so following D13 literally changes the audit's matches
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:D13
    summary: D13 leaves open how the shared unit meets a markdown span, since the audit feeds it one command string and the plan check feeds it whole fenced blocks that may hold file content, with quote blanking stateful across the block
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/discussion/decision-record.md:Constraints, PATTERN-fabric-vocabulary
    summary: The vocabulary Constraint binds the fabric repo wording to four surfaces only, while the acceptance criteria add planparser and websterengine doc text and the check's own comments that the same scan covers
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:D11
    summary: D11 does not say whether a run with an unreadable state enters the identity profile line, nor whether a run with no review files at all counts as zero findings or as not read, which decides if a figure is silently zero
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:Scope Out, D13
    summary: Scope says the audit's patterns move to the home D13 names, but D13 declares the rule on the fabric side where the patterns already live, so nothing moves and the real change is a location-free matcher returning matched text
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:Constraints, PATTERN-sole-parsers
    summary: The check is said to live in planparser over its model, but the model holds no fenced blocks or inline spans, only parsed fields, so the scan scope of D13 needs raw plan text the model does not carry
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:Acceptance criteria, per-profile summary
    summary: The per-profile acceptance criterion omits that a line states how many runs contribute findings, which D11 requires and a test must reach
---

### [MEDIUM] D13 misstates which preprocessing precedes which pattern

**Location:** `_lyx/discussion/decision-record.md`, D13 ("the shared rule is the three location-free patterns with the preprocessing that precedes them (heredoc bodies dropped, quoted spans blanked …)").
**Issue:** In `internal/fabricengine/refscanner.go` (`RefScanner.Matches`), `stripHeredocBodies` precedes all patterns, but `blankQuoted` precedes only the bare-word pattern and the spelling pattern.
The separator-prefixed path pattern runs on the heredoc-stripped text with quotes intact; the type's own comment says a path counts "quoted or not".
D13's sentence reads as if quote blanking precedes all three.
A plan writer who builds the shared unit that way makes a quoted `"/hub/x-<suffix>"` stop matching, which changes the audit's matches and breaks D13's and the Constraints' "the audit's matches do not change".
**Fix:** State the order per pattern in D13: heredoc bodies dropped first for all three; the path pattern on that text as is; the bare-word and spelling patterns on the quote-blanked copy.
Also say the matched text the finding names is taken from the copy the pattern ran on.

### [MEDIUM] D13 does not say how the shared unit applies to a markdown span

**Location:** `_lyx/discussion/decision-record.md`, D13 and the Goal.
**Issue:** The audit's unit is one recorded Bash command.
The plan check feeds it a Verify value, the verify section, every fenced code block and every inline code span.
Two consequences are not stated.
First, `blankQuoted` carries quote state across its whole input: a lone apostrophe in a fenced block (a `#` comment, a Go or YAML snippet the card tells the implementer to write) blanks everything to the block's end, so later bare words and spellings in that block are missed.
D13's Bound lists the audit's own misses but not this one.
Second, a fenced block is scanned as if every line were a command, whatever it holds.
A block that carries file content for the implementer to write is scanned for the path rule anywhere in its text and for the spelling rule after any `|`, `;` or `&`, which includes a markdown table row.
D13's rationale argues about spans that document a command; Open risks names only the inline-span path refusal.
The record leaves open whether the unit runs once per span or per file, and whether all fenced blocks count or only shell-tagged ones.
That choice decides which plans Plan-Gate bounces.
**Fix:** Add one Decision clause stating that the matcher runs per span (blanking state resets at each span) and which fenced blocks count (every block, or only those whose info string is a shell or empty).
Add the false-refusal case for a non-command block to Open risks, next to the inline-span case.

### [MEDIUM] The vocabulary Constraint covers four surfaces, the scan covers more

**Location:** `_lyx/discussion/decision-record.md`, Constraints, the `PATTERN-fabric-vocabulary` entry.
**Issue:** The entry is accurate on the mechanics it gives (`bareVocabularyToken` is a case-insensitive substring match, `vocabExempt` strips whole tokens first, the three exempt spellings are the ones D13 refuses, tokencount's non-test `.go` is unscanned) and is sufficient for the fixture questions: spelling cases come from `lyx fabric …` or the three exempt spellings; path cases come from an exempt fabric helper such as `WeftWorktree` as `internal/websterengine/audit_test.go` does; the flag name comes from a non-test constant (today `tools/tokencount/main.go` passes `"weft"` as a literal to `fs.String`, so D8 needs that constant).
It says "fabric repo" is the wording only for "the check's finding, `loom-plan-spec.md`, `refusal-spec.md` and the plan template".
The same scan covers non-test `.go` under `internal/` and every `.md` under `docs/`, `contracts/` and `internal/`.
The acceptance criteria require `internal/planparser`'s check list and doc comments, `internal/websterengine/doc.go` and the new `internal/websterengine` comments for the accept route, and `docs/` pages, all in that scope.
D13, D14 and the Goal are themselves written with "weft", which the plan writer will copy into those cards.
The failure surfaces at `go test`, so it costs a rework round and is not silent.
**Fix:** Replace the four-surface list with the scan's scope: every changed comment, string and identifier in non-test `.go` under `internal/` outside the fabric side, and every changed `.md` or `.yaml` under `contracts/`, `docs/`, `plugins/`, `crucible/`, `cmd/` and `internal/`, says "fabric repo" or a `vocabExempt` spelling.
Add one clause that a committed fixture under a `testdata` directory is not scanned (`scankit` skips it), and that only a helper already in `vocabExempt` may be named in a test.

### [MEDIUM] D11 leaves two inputs to the "never a silent zero" rule undecided

**Location:** `_lyx/discussion/decision-record.md`, D11 and Open risks.
**Issue:** The set of runs that contribute findings is named (one profile, not restarted with `--fresh`, review files read), and a profile line with none shows not available; those two edges are settled.
Two inputs to the figures are not.
First, "a run with no recorded partition counts as `identity`" and "a run whose state cannot be read keeps its row with the column marked" collide for a run whose last webster state is unreadable: it has no recorded partition, so it is not stated whether it enters the `identity` line.
If it does, its failed batches and recoveries enter that line as zero because nothing could be read, which is the silent zero the criterion forbids.
Second, "review files were read" does not say whether a weft history with no `_lyx/reviews/webster/` content at all (a run that ended before any review round, or an older shape, which Open risks says exists) counts as read with zero findings or as not read.
The first makes findings per card zero for that run; the second leaves it out of the contributing set.
**Fix:** State in D11 that a run whose last state cannot be read enters no profile line and is listed with the reason, and that a history with no review round files is read with zero findings only when the state shows a Webster-Review round ran, otherwise it is marked not read.

### [LOW] Scope's "moving its patterns" contradicts D13's "declared once on the fabric side"

**Location:** `_lyx/discussion/decision-record.md`, Scope Out ("moving its patterns to the home D13 names keeps what they match") and D13.
**Issue:** The audit's patterns already live in `internal/fabricengine/refscanner.go`, which is the fabric side D13 names as the home.
Nothing moves.
What changes is that the three location-free patterns are exposed as a matcher that needs no worktree location, separates the path rule from the spelling rule, and returns the matched text; today's seam returns a bool from a type built over a location.
**Fix:** Reword the Scope Out clause to say the audit keeps its matches while the location-free part is exposed to the plan check.

### [LOW] The check is said to run over planparser's model, which does not hold the scanned spans

**Location:** `_lyx/discussion/decision-record.md`, Constraints (`PATTERN-plan-generation` and `PATTERN-sole-parsers` entry) and D13.
**Issue:** `Plan` and `Card` (`internal/planparser/plan.go`) hold parsed fields: `Verify`, `Intent`, `Framing`, `SharedDecisions`, `RenameMechanic` and the overview's `Verify` joined with " && ".
They hold no fenced blocks or inline spans, and a card file's lines outside its labelled fields are not retained, yet D13 scans "every plan file, cards and overview alike".
The check therefore needs either a model field for the raw text or a re-read of the files, as `cardHeadingNumber` does.
**Fix:** Say in the Constraint that the check reads the plan files' raw text inside `planparser` (by a model addition or a re-read) and that it stays the sole parser.

### [LOW] The acceptance criterion drops the contributing-run count

**Location:** `_lyx/discussion/decision-record.md`, Acceptance criteria, the per-profile summary bullet.
**Issue:** D11 says a profile line states how many runs contribute findings.
The criterion states the denominator and the not-available display but not the count, so a test written from the criteria alone need not reach it.
**Fix:** Add "and states how many runs those are" to the criterion.
