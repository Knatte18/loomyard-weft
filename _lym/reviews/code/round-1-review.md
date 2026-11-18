---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/shellguard/rows_test.go#TestRows_SteersOpenWithWholeCommandRefusal
    summary: The redundancy report lists this test as contained, and it is kept for assertions its covering tests lack, but it has no testtiming keep marker as PATTERN-test-speed requires.
    fix: In internal/shellguard/rows_test.go, add the line //testtiming:keep followed by a reason directly above func TestRows_SteersOpenWithWholeCommandRefusal. The reason names what the test pins that TestCheckTexts and TestRows_EngineTexts do not, for example "//testtiming:keep pins that every row's steer on claude and codex opens with the whole-command clause and that no notice carries it". Leave the test body and its doc comment unchanged.
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: consistency
    location: internal/shellguard/rows.go#pythonRoutes
    summary: The comments on pythonRoutes, shellEditRefusal and headlessClaudeRefusal still say each one opens every steer of its row, but every shipped steer now opens with wholeCommandRefusal.
    fix: In internal/shellguard/rows.go, reword the doc comments on pythonRoutes (line 81), shellEditRefusal (line 93) and headlessClaudeRefusal (line 181) so that each says it opens the row's own steer text, which follows the shared whole-command clause. For example, write "pythonRoutes opens the python row's own steer text, the refusal and the parse routes in Python's place", and reword the other two the same way. Change no code.
    origin: assist:assist
---

### [BLOCKING] Contained test kept without its keep marker

**Location:** internal/shellguard/rows_test.go#TestRows_SteersOpenWithWholeCommandRefusal

**Issue:** The gate's redundancy report lists `TestRows_SteersOpenWithWholeCommandRefusal` as contained by `TestCheckTexts` and `TestRows_EngineTexts`.
I compared their assertions:

- `TestCheckTexts` checks the clause only on the Codex python steer (`rows_test.go:54`).
- `TestRows_EngineTexts` checks Claude tool names, the Codex edit rule and the Claude fallback (`rows_test.go:124-141`), and never the clause.

The new test is the only one that asserts the clause on every row for both engines, the refuse-raw row included.
It also asserts that the clause is absent from every notice, and nothing else checks this for the Codex shell-edit and headless-claude notices.
So the test should be kept, not folded or deleted.
`pattern/PATTERN-test-speed.md` says that a kept candidate carries `//testtiming:keep <reason>` directly above its `func Test…` line.
This test has no such marker.
The prompt's constraints rule says that violating a listed PATTERN entry is BLOCKING.

### [LOW] Fragment comments still claim to open every steer

**Location:** internal/shellguard/rows.go#pythonRoutes

**Issue:** `openSteersWithWholeCommandRefusal` now puts `wholeCommandRefusal` in front of every shipped steer.
The new comment on `wholeCommandRefusal` (`rows.go:380`) says that it "opens every steer, plain and per engine".
Three older comments in the same file still claim the opening position for their own fragments:

- `pythonRoutes opens every python steer` (`rows.go:81`)
- `shellEditRefusal opens every shell-edit steer` (`rows.go:93`)
- `headlessClaudeRefusal opens every headless claude steer` (`rows.go:181`)

The file now contradicts itself about what opens a steer.

## Holistic review

- **Fasit.** The board entry's Idea asks for three things, and the diff meets each:
  - **The refusal says nothing ran.** The diff applies one clause in `Rows` to every plain and engine steer.
  - **It names the refused segment.** Each row's own text follows "The refused part: " and names the refused command by kind.
  - **The writer first checked the current text.** The summary says no old steer said that nothing ran, and the base constants confirm it.
- **Golden fixture.** With the clause stripped, the HEAD `settings-golden.json` is byte-identical to the base, so it has no unrelated drift.
- **Commits and summary.** The commit messages and `_lym/landing/summary.md` match the diff. No `Pattern-Departure:` trailer is present, and none is needed.
- **PATTERN fit.** The clause lives inside `internal/shellguard`'s table, and both engines render it through `Row.Steer`/`SteerFor`, as `PATTERN-shell-guard-table` requires. No other listed entry is touched.

## Rejected

- The refused part is named by row category rather than quoted from the command, which the code-scope lens flagged as a reading of "names the refused segment". This is not a defect. The static `grep -E` match cannot isolate the segment safely, as the summary explains, and each row's text identifies which part of a compound command was refused.
- The assistant rated F1 BLOCKING and the code-tests lens rated it LOW. I merged them at BLOCKING because the missing marker breaks a listed PATTERN rule, and the constraints section makes that BLOCKING.
- The empty-engine-steer branch in `openSteersWithWholeCommandRefusal` has no test, which the code-tests lens noted without raising a finding. This is not a coverage gap: no shipped row reaches the branch, and `Rows` takes no row data from callers.
