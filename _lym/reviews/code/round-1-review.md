---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/burlerengine/assist_test.go:283
    summary: New tests assert the static prose of the assistant stencils, which PATTERN-test-speed forbids
    fix: In TestEngine_Review_AssistantPromptFollowsItsFan (assist_test.go) delete the wantInText prose strings ("You work alone - no other session works with you.", "are part of your own work and are no other session", the solo "You work alone - you write your output and nothing else.") and the wantAbsent strings ("you write your output and nothing else", "Your output is the only file set you write", "subagent", "lens"). Pin which stencil Go chose through values Go fills instead - the fanned row wants lensreport.PathPattern(report) and lensreport.Path(report, "style") in the prompt, the solo row wants neither. In internal/seatengine/prompt_test.go delete the same prose strings from the AFannedAssistantIsToldItsSubagentsAreItsOwnWorkAndItsReportsAreItsToWrite subtest and the "you write your output and nothing else" absence check, and keep the Go-filled output_files list assertion (seat.Outputs[0] followed by the two lens paths).
    origin: lens:code-tests
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/shedadapters/burlerreview.go:425
    summary: assist_ending and assist_lenses_missing survive a later attempt of the round that ran no assistant
    fix: In acceptedExit assign u.AssistEnding and u.AssistLensesMissing unconditionally - clear both to their zero values, then set them from result.Assists - so a record never keeps an earlier attempt's ending beside assist_fan empty. Extend the recordRoundAttempt doc, which says it is the only writer of the three keys, only if recordRoundAttempt is also made to reset them. Add a row to TestBurlerReviewProducer_Call_RecordsAssistOutcome that pre-seeds the round's usage record with assist_ending done and a missing lens, then accepts a round with no Assists, and asserts both keys read empty.
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/burlerengine/review.go:242
    summary: Nothing pins that Review hands the assistant's fan to keepAssistReports, so a fanned run could stop reporting missing lenses and every test would pass
    fix: In TestEngine_Review_AssistantPromptFollowsItsFan, whose fanned row runs Review with AssistFan standard and a shuttle that writes no lens file, keep the ReviewResult Review returns and assert result.Assists has one outcome whose MissingLenses equals the fan's lens names in fan order for the fanned row and is empty for the solo row. TestKeepAssistReports_MissingLenses stays for the file-reading cases, and its table rows need no change.
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: design
    location: internal/lensreport/lensreport.go:50
    summary: Returned searches the whole file for one exact byte string instead of reading the outcome key of the opening block
    fix: Make Returned read only the file's opening fenced yaml block and compare the trimmed value of its outcome line with reported, using the standard library only so the leaf-package rule holds. A missing or malformed block, an unreadable file and an outcome of no report return false. Extend TestReturned with CRLF line endings, trailing whitespace after reported, and a file with no block whose message holds the line outcome - reported.
    origin: lens:code-correctness
  - id: F5
    severity: LOW
    class: consistency
    location: internal/seatengine/table.go:35
    summary: Three comments still say every assistant includes seat-directive-assistant or removes only markers and outputs, which the fanned variant makes false
    fix: In table.go reword the assistantDirectiveBlock comment so it is the block a solo assistant's stencil includes. In internal/seatengine/doc.go:112 reword "every assistant's seat-directive-assistant" to a solo assistant's, with the fanned block named on the line the diff added. In internal/seatengine/engine.go:244 add subagent reports to what Run removes before the chair starts.
    origin: lens:code-affected-set
  - id: F6
    severity: LOW
    class: consistency
    location: internal/seatengine/prompt_test.go:329
    summary: The seatengine fanned subtest and the burlerengine fan test both assert that the spec outputs are the report alone and that the lens paths are listed
    fix: Delete the seatSpec OutputFiles assertion at the end of the AFannedAssistantIs... subtest in prompt_test.go; TestEngine_Review_AssistantPromptFollowsItsFan asserts the same report-only outputs through Review. Keep the output_files list assertion there, since it pins the seatengine marker.
    origin: lens:code-tests
---

### [BLOCKING] New tests assert the static prose of the assistant stencils, which PATTERN-test-speed forbids

**Location:** internal/burlerengine/assist_test.go:283, internal/seatengine/prompt_test.go:343

**Issue:** `PATTERN-test-speed` under Test economy says no test pins a stencil's static prose, and lets a stencil test pin only a marker set, a Fill round-trip, a value Go fills or reads, a fragment Go chooses, or a rule across a stencil family.
`TestEngine_Review_AssistantPromptFollowsItsFan` asserts the sentences "You work alone: no other session works with you.", "are part of your own work and are no other session" and "You work alone: you write your output and nothing else.", and asserts the absence of "you write your output and nothing else", "Your output is the only file set you write", "subagent" and "lens".
The seatengine subtest `AFannedAssistantIsToldItsSubagentsAreItsOwnWorkAndItsReportsAreItsToWrite` asserts the first two sentences and the same absence.
The choice between the solo and the fanned stencil is Go's, and the lens report paths Go fills into the prompt already show it, so no wording assertion is needed to pin it.
The wording is review's to guard, and a reworded sentence in either directive breaks these tests without any behavior changing.
The severity follows the constraints directive: a violation of a listed constraint is BLOCKING.
The lens report path and `output_files` assertions in the same tests are values Go fills and stay.

### [MEDIUM] assist_ending and assist_lenses_missing survive a later attempt of the round that ran no assistant

**Location:** internal/shedadapters/burlerreview.go:425, internal/shedadapters/roundusage.go:86

**Issue:** `recordRoundAttempt` resets `Attempt`, `Assist` and `AssistFan` at each attempt but not the two new keys, and `acceptedExit` writes them only inside a loop over `result.Assists`.
A round accepted once with a fanned assistant and accepted again with none, for instance with the assistant turned off in config between attempts, keeps `assist_ending: done` and the earlier `assist_lenses_missing` beside `assist: nil` and `assist_fan: ""`.
The record exists so a seat that went solo cannot read as a fanned one, and this leaves it naming an assistant that did not run.
`TestBurlerReviewProducer_Call_RecordsAssistOutcome` starts every row from a fresh run directory, so it never reaches the case.

### [MEDIUM] Nothing pins that Review hands the assistant's fan to keepAssistReports

**Location:** internal/burlerengine/review.go:242

**Issue:** `concludeReview` passes `e.assistLensNames(opts.AssistFan)` to `keepAssistReports`, and that argument is the link between the fan a round asks for and the lens the record names as missing.
`TestKeepAssistReports_MissingLenses` calls the helper with the names handed in, `TestEngine_Review_AssistantPromptFollowsItsFan` discards the result `Review` returns, and `TestBurlerReviewProducer_Call_RecordsAssistOutcome` feeds preconstructed outcomes.
Passing `nil` there would leave every test green while no fanned run ever listed a missing lens, which is the second bullet of the Idea.
The assistant's report raised the same gap as a BLOCKING repository-rule violation of the helper-level test; it is kept here as an uncovered behavior, since `TestKeepAssistReports` already tests the helper directly and predates this diff.

### [LOW] Returned searches the whole file for one exact byte string instead of reading the outcome key of the opening block

**Location:** internal/lensreport/lensreport.go:50

**Issue:** The spec and `PATTERN-sole-parsers` say `Returned` reads the `outcome` key alone, but it runs `bytes.Contains` for `\noutcome: reported\n` over the whole file.
A spec-shaped report written with CRLF line endings, or with trailing whitespace after `reported`, reads as not returned and its lens is recorded as missing although it ran.
A file with no opening block whose message holds the line reads as returned.
Both need an agent that departs from the format, so the effect is a false record in a rare case.

### [LOW] Three comments still say every assistant includes seat-directive-assistant or removes only markers and outputs

**Location:** internal/seatengine/table.go:35, internal/seatengine/doc.go:112, internal/seatengine/engine.go:244

**Issue:** The `assistantDirectiveBlock` comment says it is the block every assistant seat's stencil includes, and the package doc says the same at line 112.
The diff adds, at doc.go line 115 and in `fannedAssistantDirectiveBlock`, that a fanned assistant includes the fanned block in its place, so each pair of statements contradicts itself.
The `Run` comment lists a fresh run's removals as end markers and outputs, while `clearAssistantFiles` now removes subagent reports too.

### [LOW] The seatengine fanned subtest and the burlerengine fan test both assert that the spec outputs are the report alone and that the lens paths are listed

**Location:** internal/seatengine/prompt_test.go:329

**Issue:** `TestEngine_Review_AssistantPromptFollowsItsFan` asserts the shuttle spec's `OutputFiles` is the report alone and that the composed prompt lists the lens report paths, through the real stencils.
The seatengine subtest asserts the same `OutputFiles` through `seatSpec`, and the same listing through a test stencil.
The `output_files` order is the seatengine behavior and stays in its subtest; the spec outputs assertion is covered by the burlerengine test.

## Rejected

- Assistant F1 as a BLOCKING rule violation by `TestKeepAssistReports_MissingLenses`: the helper is already tested directly by `TestKeepAssistReports`, which predates the diff, so the new table is no new per-helper test; the real gap, the unpinned wiring, is kept as the MEDIUM finding above.
- Assistant F1's fix to move every missing-lens case into `TestEngine_Review_AssistedRound` and delete the helper test: the file-reading cases (absent, `no report`, non-done ending) are cheap at the helper and are not duplicated by a public-surface test.
- Assistant F2 as MEDIUM: kept as LOW, since both directions need a lens report that departs from the pinned format.
- Code-tests finding on an unknown fan keeping the seat solo: the seat does not start when its fan is unknown, so its stencil is never read; no behavior is left unpinned.
- Code-tests finding on a probe or resume row with a fan: `probeTable` and `Review` share `assistSeat`, and the resume link to `keepAssistReports` is the wiring finding above.
- Code-tests finding on pre-seeding a record: kept as part of the MEDIUM staleness finding, not as its own.
- The request for a real fanned seat in which every lens starts and returns: the diff cannot supply it, and `_lym/landing/summary.md` says so under "Not yet proven".
