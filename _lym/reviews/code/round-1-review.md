---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/shedverbs/status_test.go:699
    summary: The one-shot status test stub gives Text and Steady the same value, so a swap to Steady in statusCmd would pass
    fix: In TestStatusCmd_WaitingNote_RunningOnly, make the hook return a WaitNote whose Steady differs from its Text, and assert the envelope's waiting key and the human view carry Text.
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: design
    location: internal/loomcli/checklist.go:169
    summary: No test drives checklistPoll, so the pane keeping the elapsed time through note.Text is unpinned
    fix: Add a case that calls checklistPoll with a Waiting hook returning distinct Text and Steady, and asserts the frame's wait line shows Text.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: design
    location: internal/shedverbs/status.go:122
    summary: The unavailable branch of statusWatchLine returns the line as its own key with no assertion on the key
    fix: In TestStatusWatchLine_AppendsWaitingNote or a sibling test, point the spec at a missing status file and assert line == key == UnavailableLine(label).
    origin: lens:code-tests
---

### [LOW] One-shot status test cannot tell Text from Steady

**Location:** internal/shedverbs/status_test.go:699
**Issue:** The Goal says the one-shot status keeps the elapsed time. `TestStatusCmd_WaitingNote_RunningOnly` returns `WaitNote{Text: note, Steady: note}`, so a regression in `statusCmd` that put `waiting.Steady` into the envelope or the human view would still pass.

### [LOW] The checklist pane's use of Text is untested

**Location:** internal/loomcli/checklist.go:169
**Issue:** `checklistPoll` takes `note.Text`, which keeps the elapsed time in the pane. Only `renderChecklist` is tested, with a literal `waitNote`. Switching to `Steady` would put `<elapsed>` in the pane and fail no test.

### [LOW] The unavailable key has no assertion

**Location:** internal/shedverbs/status.go:122
**Issue:** `statusWatchLine` returns `unavailable, unavailable` when the status file cannot be read. `TestUnavailableLine_DedupesAcrossPolls` only checks the line is byte-identical across calls. Nothing pins that the key equals the line.

## Rejected

- Lens code-tests F-D (NIT, `steadyOf` derives the expected Steady from Text): the helper fails when Steady keeps an elapsed time or changes any other part, and the only elapsed strings in play are `6m` and `3m`, so the doubt is not a defect.

## Holistic notes

The diff matches the Goal.
Every elapsed rendering in a note's Text is replaced in its Steady form, including the running-verify, shuttle and combined gate-plus-rest notes.
Holders, attempt and detail text stay in Steady, so a real change still prints.
The envelope's `waiting` key, the status file schema and the human view carry Text and are unchanged.
The spec, overview, verb help and flag text match the behavior, and no commit carries a `Pattern-Departure:` trailer.
`lym gate test ./internal/shedverbs ./internal/loomcli` answered with a carried green record: 0 tests discovered, exit 0.
