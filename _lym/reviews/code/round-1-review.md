---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/shuttleengine/claudeengine/settings.go:441
    summary: The batching sentence tells every role to keep a read file in mind rather than read it again, with no exception for a file changed since, which the Webster stencil carries.
    fix: In the `batchingSentence` constant, qualify the clause, for example "keep a file you have read in mind rather than reading it again unless something other than your own edit has changed it", keeping the text on one line with no quote, backslash or line break. The tests and `doc.go` build on the constant, so no other edit is needed.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:419
    summary: The shuttle paragraph still describes the appended sentence as only asking to batch calls in one message, while the sentence now also asks to read a file whole or all its spans at once and keep it in mind.
    fix: In the sentence of the shuttle paragraph that says every Claude session "also gets the creel trust sentence and a sentence asking it to request every call whose arguments it already knows in one message", add that the sentence also asks it to read a file whole or all its spans in one message and not to read it again, and end the clause "so a session knows ... and batches independent calls" with "and reads each file once".
  - id: F3
    severity: LOW
    class: consistency
    origin: handler
    location: internal/shuttleengine/claudeengine/settings.go:440
    summary: The doc comment on `batchingSentence` says it only tells sessions to request independent calls together, but the sentence now also governs how files are read.
    fix: Reword the comment above `batchingSentence` to state both asks, for example "batchingSentence tells every Claude session shuttle launches to request independent calls together and to read a file whole, or every span it needs, in one message and not again, since a session's cost grows with its turns times its context."
---

### [LOW] Batching sentence drops the re-read exception

**Location:** internal/shuttleengine/claudeengine/settings.go:441
**Issue:** The stencil at `contracts/stencils/webster/webster-body-implementer.md:55` allows a re-read where a change not made by hand touched the file. The system-prompt sentence, which every Claude session with a run directory receives, says only "keep a file you have read in mind rather than reading it again". Roles that read files changed by forks, tools or other sessions get the unqualified rule. The Edit tool's staleness error bounds the harm, so this is an over-statement and not a break.

### [LOW] docs/overview.md still describes the old batching sentence

**Location:** docs/overview.md:419
**Issue:** The paragraph says the appended sentence asks a session to request every call whose arguments it already knows in one message, so a session batches independent calls. `doc.go:225` was updated for the added read rule and this description was not. The text is still true but incomplete.

### [LOW] Doc comment of `batchingSentence` omits the new ask

**Location:** internal/shuttleengine/claudeengine/settings.go:440
**Issue:** The comment names only the request-independent-calls-together purpose. The constant now also tells the session to read a file whole or by all its spans in one message and to keep it in mind. A reader judging fitness from the comment misses half the sentence's content.

## Rejected

- The NIT from the code-correctness lens that the stencil's re-read exception does not name a formatter or guard-step rewrite: "such as" in `webster-body-implementer.md:55` makes the list open, and the case self-corrects through the Edit staleness error.
- No finding on scope: the diff changes the Webster implementer stencil, the batching sentence and its `doc.go` paragraph, which are the two things the board entry asks for plus the doc that describes one of them. The entry's `edit-with-edit-tool` note asks for no change, and the diff drops the stencil's "or reading whole files" clause that note makes obsolete.
- No finding on the summary: each claim under "What changes" matches the diff. Its remark about the `batching-in-claude-local` run and heddle's `shipped.json` concerns another run and is not checkable from this diff.
- No finding on tests: the delivery tests build their expectation from the constant, and an assertion on the new wording would be a new assertion on static prose.
- Stencil wording: no rule the entry did not ask to change is dropped. The read-spans-before-first-edit and batch-independent-edits rules both remain, at lines 54 and 56, and the new wording agrees with the quarry-span line above it and with the retired-name sweep below it.
