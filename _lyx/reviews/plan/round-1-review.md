---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/03-vocab-root-replaceall.md:8
    summary: Card 3 lists internal/lyxcwd/vocabscan_test.go as a plain test-file path while its Intent adds rows to the existing TestFabricVocabularyFailures_Fixtures, which the plan stencil requires as a member glyph
    fix: In _lyx/plan/03-vocab-root-replaceall.md, under **Edit:**, replace the bullet `internal/lyxcwd/vocabscan_test.go` with `internal/lyxcwd#TestFabricVocabularyFailures_Fixtures` (resolves to internal/lyxcwd/vocabscan_test.go:27-58). The validate-plan --overlay run with this change reports ok.
  - id: F2
    severity: NIT
    class: consistency
    location: _lyx/plan/02-reader-paren-dollar.md:37
    summary: Card 2 says each new test row reaches readOnlyReaderSegment, but the git diff *.go and git diff -- *.go rows are git segments that never reach it
    fix: 'In _lyx/plan/02-reader-paren-dollar.md, replace the line that introduces the test rows (it ends in "each reaching `readOnlyReaderSegment` through `IsReadOnlyCommand` after every command-wide scan passes:") with "Test rows join the table of `TestIsReadOnlyCommand` in `internal/fabricengine/refscanner_test.go`; each reader row reaches `readOnlyReaderSegment` through `IsReadOnlyCommand` after every command-wide scan passes, and the two `git diff` rows reach the git form check:". The validate-plan --overlay run with this change, together with F1, reports ok.'
---

### [MEDIUM] Card 3 names the extended fixture test by file path, not member glyph

**Location:** `_lyx/plan/03-vocab-root-replaceall.md:8` (`**Edit:**` bullet `internal/lyxcwd/vocabscan_test.go`)

**Issue:** The card's Intent adds two rows to the table of `TestFabricVocabularyFailures_Fixtures`, an existing test function in `internal/lyxcwd/vocabscan_test.go`.
The plan stencil (`loom-template-plan.md`, the `**Edit:**` rules) says a card names an existing test function it changes as a member glyph, and keeps a plain test-file path only for a test file the card adds tests to without changing an existing member.
The rubric's member-glyph check flags this case.
`lyx quarry resolve internal/lyxcwd#TestFabricVocabularyFailures_Fixtures` finds it at `vocabscan_test.go:27-58`.
The card's other `Edit` member, `internal/lyxcwd#fabricVocabularyFailures`, resolves into `enforcement_test.go`, so the swap does not trigger `redundant-file-target`.

### [NIT] Card 2's reachability sentence covers the git rows too

**Location:** `_lyx/plan/02-reader-paren-dollar.md:37`

**Issue:** The sentence introducing the test rows says each row reaches `readOnlyReaderSegment` through `IsReadOnlyCommand`.
The `git diff *.go` (false) and `git diff -- *.go` (true) rows are `git` segments.
`readOnlySegment` sends them to `readOnlyGitSegment` and `gitForm.admits`, never to the new reader function.
The reader rows do reach it as claimed (see below), so this is wording only.

## Rejected

- Card 3 `Edit` targets in `enforcement_test.go`: the dropped `if vocabScanRootFiles[rel]` block (lines 629-632) and the doc comment both lie in `internal/lyxcwd#fabricVocabularyFailures`'s span (582-649).
  `internal/lyxcwd#recordsRepositoryName` resolves to a const at `enforcement_test.go:570-571`, and no other `.go` file in the module references it.
  `strings` stays used in that file.
  No finding.
- Card 3 verify coverage: `enforcement_test.go` and `vocabscan_test.go` carry no build tag, so `go test ./...` runs both.
  So does `TestEnforcement_FabricVocabulary`, the test that pins the current-tree scan.
  The only tagged file in `internal/lyxcwd` is `lyxcwd_test.go`, which the card does not touch, and `go test -tags integration ./...` runs it anyway.
  The `## verify:` order matches the stencil: build, vet under integration, tmux and llm, `go test`, then `go test -tags integration`.
  No finding.
- Card 1 test economy: no existing test covers `-no-cutoff`, since the flag does not exist.
  `TestRunRefuses` in `main_test.go` covers refusals only.
  Extending `TestRunReadsArchivedShedStatus` (integration tier, the one test that drives `run` over real archive tags) is the extension `PATTERN-test-economy` asks for.
  The fixture arithmetic holds: `early`'s Plan row runs from the seed's 08:30 to 10:03, which is `1h33m0s`, against `1h3m0s` with the 09:00 clamp.
  `recentRuns` skips a slug with no `*.jsonl`, so adding `early`'s tags before the default run leaves its skipped header unchanged.
  The without-flag case for an untimestamped transcript is pinned in `count_test.go` (the bounded `CountRun` case with `n-none.jsonl`).
  No finding.
- Card 1 `Edit` members and the `main.go` package doc comment: the doc comment (lines 1-158) lies outside `tools/tokencount#run`'s span (182-307), and the test file's header comment lies outside the test's span (21-175).
  The stencil gives `**Edit:**` member glyphs for the changed function and test.
  It allows a file path only for a file with no symbol to name, and `redundant-file-target` refuses a file self glyph beside a member of that file.
  The Intent names both comment locations explicitly.
  No finding under the symmetry rule.
- Card 2 reachability: I traced each prescribed false row through `IsReadOnlyCommand`'s command-wide scans (substitution markers on the single-quote-blanked view, `>`, `&`).
  Each reaches `readOnlySegment` intact as a reader segment, and only the parenthesis or dollar rule rejects it.
  For `cat $'\'' $(touch x) f`, `blankQuotedSpans` closes the span at `\'`'s quote and opens an unterminated one at the next `'`, hiding `$(` and the `(`.
  Only the dollar rule then fires, on `$` followed by a raw `'`, as the card states.
  The true rows `jq '.a | map(.b)' f.json`, `grep a$ f`, `grep 'a$' f` and `grep "^a$" f` pass both rules.
  No upstream check rejects a named input.
- Card 2 attack surface: the change only narrows read-only, and the Intent's rule definition states what still passes: a `$` followed by whitespace, a double quote or the segment's end.
  It also states the bound: such a command fails the read-only check and the webster audit treats it as any other.
  This is D2's own rule, so no finding.
- Card 2 ImpactSummary: the callers are `webstercli` (`acceptaudit.go`, `recordbatch.go`, `recoverbatch.go`, `run.go`) and `loomcli/wiring.go`.
  No test under `websterengine`, `webstercli` or `loomcli` feeds a reader command holding `$`, `(` or `)`.
  The claim holds.
- Card 2 test rows `git diff *.go` and `git diff -- *.go`: the second pins `diff`'s own `globAfterDoubleDash` entry in `gitForms`, which no existing row covers.
  The first shares the pre-`--` glob branch with `git log *`, but both are table rows the acceptance criterion names, not a new test function.
  No test-economy finding.
- Fidelity: D1 (flag, zero bound, `Older` cleared, header line, doc-comment description with the `-calibrate` caveats and untimestamped rule) is carried by card 1.
  D2 (both rules, doc comment, `doc.go` paragraph with reader rationale, admission rule and git `--` reason) is carried by card 2.
  D3 (constant and `ReplaceAll` removed, doc comment keeps the name, fixture rows) is carried by card 3.
  The Constraints are met: comment line breaks are stated on cards 1 and 2, tests extend existing tables and tests, the tokencount test runs at the integration tier as the record's notes direct, and the classifier design goes in `doc.go`.
  The header line text and the `doc.go` paragraph content restate D1 and D2 and add no scope.
- Card order: card 1 uses only `tools/tokencount` symbols, card 2 only `fabricengine` symbols, and card 3 only `lyxcwd` test symbols.
  No card uses a symbol another card creates or changes.
