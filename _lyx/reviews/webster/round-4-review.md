---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/shuttleengine/attach.go:281
    summary: the offset-seeding sentence has a rewritten tail clause and still breaks mid-phrase across three lines
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/shuttleengine/attach_test.go:904
    summary: the TestAttach_OffsetStartsAtZero doc sentence has a rewritten tail clause and still breaks mid-phrase across four lines
  - id: F3
    severity: BLOCKING
    class: consistency
    location: internal/treadleengine/engine_test.go:261
    summary: the TestEngine_RetrySemantics doc sentence has a rewritten tail clause and still breaks mid-phrase after attempt
  - id: F4
    severity: BLOCKING
    class: consistency
    location: internal/shuttlecli/smoke_guardrail_test.go:3
    summary: the file-header sentence has a rewritten tail clause and still breaks mid-phrase across seven lines
  - id: F5
    severity: BLOCKING
    class: consistency
    location: internal/shuttleengine/claudeengine/settings.go:1
    summary: the file-header sentence has a rewritten tail clause and still breaks mid-phrase across nine lines
---

Range: no `_lyx/loom/rework/` round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD` (`product.parent` is `main`).
Method: every `//` line the diff adds in Go files (414 lines, 237 blocks, derived mechanically from `git diff -U0`) was grouped into blocks and read with its removed counterpart, then the added lines that end without sentence-final punctuation were listed and read in their file context.
Added `//` lines in new files, test files and `doc.go` are all included.

Verdicts on the focus questions:

- **Added sentences.**
  Every sentence the diff adds as new text (new files, new doc comments, new field comments, new blocks) sits on one line per sentence, with further breaks only at `;`, `:`, `—` or a `, so`/`, and` clause boundary, or between the items of a coordinated list or compound predicate.
  Breaks between coordinated predicates or list items (`shuttleengine/doc.go:62-66`, `hold.go:39-42`, `claudeengine/doc.go:74-75`) are between phrases, not inside one, and each line is a full phrase, so they are not a defect.
  No added line carries two sentences except three legacy lines that only swapped or lost words (`shuttlecli/run.go:34`, `shuttleengine/spec.go:31`, `shuttleengine/spec_test.go:18`), which are exempt.
- **Rewritten tails.**
  Edits that only swapped or deleted words in an untouched wrapped sentence are exempt, and that covers the `asking` to `died`/`held` swaps and the `triage`/`AwaitOperator` deletions across `treadleengine`, `shedadapters`, `burlerengine`, `loomcli`, `webstercli` and `websterengine`.
  A sentence whose head the diff merely re-broke onto its own line, with no word of the sentence changed, is also exempt, because the legacy wrap of an untouched sentence is not rewrapped (`spec.go:147`, `rundir.go:87`, `shuttleengine/doc.go:54-56`, `loomcli/driverspec.go:49`, `shedadapters/singlellm.go:142`, `treadleengine/judge.go:3`, `judgeverdict.go:3`, `template_test.go:62`).
  The five findings below are the edits that replaced a clause with new words inside a sentence that is still wrapped mid-phrase.

### [BLOCKING] Offset-seeding sentence split mid-phrase
**Location:** `internal/shuttleengine/attach.go:281`
**Issue:** the diff rewrote the sentence's tail ("is correct in both AwaitOperator modes" became "is read as a held turn end"), so the whole sentence must satisfy `PATTERN-comment-line-breaks`.
It still runs over three lines and breaks mid-phrase after "seeding at EOF would" and after "converting a".
**Fix:** reflow to one clause per line: the colon head, then "seeding at EOF would mean ... OutcomeTimeout failure —", then "and a replayed backlog ending in an ask is read as a held turn end."

### [BLOCKING] TestAttach_OffsetStartsAtZero doc sentence split mid-phrase
**Location:** `internal/shuttleengine/attach_test.go:904`
**Issue:** the tail ("classifies OutcomeAsking" became "is a held turn end that keeps the run polling to its deadline") is new text, and the sentence still breaks mid-phrase after "a" and after "the first".
**Fix:** break after the colon, and at the comma before "and the same backlog".

### [BLOCKING] TestEngine_RetrySemantics doc sentence split mid-phrase
**Location:** `internal/treadleengine/engine_test.go:261`
**Issue:** the last clause was replaced wholesale ("an asking outcome's triage call determines ..." became "a died or timed-out first attempt is retried once"), and the sentence breaks between subject and predicate after "a second consecutive non-done attempt".
**Fix:** break after the colon, then keep each of the two clauses on its own line.

### [BLOCKING] smoke_guardrail_test.go header sentence split mid-phrase
**Location:** `internal/shuttlecli/smoke_guardrail_test.go:3`
**Issue:** the final clause ("surfaces it as the run loop's classified asking outcome" became "stays held, notifies its parent once and finishes once answered") is new text, and the sentence still breaks mid-phrase at "guardrail / path", "probed / (docs", "by / the", "steered / instruction" and "REAL claude / asked".
**Fix:** reflow to the colon-and-comma boundaries: the opening clause, the parenthetical, the first REAL claude clause, then "and a REAL claude asked to pose a question stays held ...".

### [BLOCKING] settings.go header sentence split mid-phrase
**Location:** `internal/shuttleengine/claudeengine/settings.go:1`
**Issue:** the final clause ("so the run loop can classify it as a real-time asking signal instead of waiting for the timeout" became "so the run loop holds it like any turn end without output") is rewritten, and the sentence still breaks mid-phrase at "a Stop / hook", "(the only channel ParseEvents / reads)" and "the / in-process Agent tool", "(where there is no operator present to answer / it)".
**Fix:** one line per list item of the guardrail list, each ending at a comma, `;` or `—`, with no break inside an item.
