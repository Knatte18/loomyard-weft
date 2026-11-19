---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/gatecli/cardfixer.go:135
    summary: The card fixer never reaches the new held-turn notifier and still launches without a parent.
    fix: "Wire parent resolution into cardFixerSpec in internal/gatecli/detachfixer.go and pass PromptInputs.ParentName. Change the cold-fixer wait path in internal/gatefixer/await.go so a questioning turn notifies the parent and remains available for its answer instead of immediately stopping as DeadReasonAsking. Exercise that actual wait path with controlled events and a fake notifier, asserting parent delivery, program identity, and continuation after the answer; a SetNotifier source assertion alone is insufficient. Update the affected gate-fixer contracts and existing tests."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/loomcli/rownotice.go:37
    summary: An invalid advisory subscription halts the run contrary to PATTERN-refusal-way-forward.
    fix: "In watchRows, warn and record an unknown notify_parent_on name with the recipe's available rows, ignore invalid subscriptions, and still wrap valid subscriptions. Remove the notification-only refusal from newShed. Update the existing unknown-row test and the refusal promises in the template, orch role, and package documentation to describe the warning and continued execution."
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:code-tests
    location: internal/loomcli/rownotice_test.go:55
    summary: Helper-only tests leave subscription activation, run lifecycle, and notification failure contracts unchecked.
    fix: "Move or extend the existing row-notice behavior tests through loom's run construction and shed stepping with fake producers and notification delivery. Load a nonempty notify_parent_on value, assert notification before producer execution, and cover skipped rows, yield, pause/resume, and loop re-entry using real persisted history. Extend the existing failure cases with status/record read and write failures, two calls after a failed post with one notification attempt, and a producer returning a populated output and sentinel error. Assert unchanged delegation rather than adding per-helper tests."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-affected-set
    location: contracts/stencils/parent/parent-directive-operator-ban.md:4
    summary: Burler role-specific messaging bans still prohibit the parent questions this directive requires.
    fix: "Amend the messaging restrictions in contracts/stencils/burler/burler-template-fixer.md, burler-template-reviewer.md, and burler-template-reviewer-assisted.md to permit open questions to the run's parent and receipt of its answer. Preserve the prohibition on reviewer/fixer communication and independent-review collaboration. Audit the corresponding assistant role restriction for the same exception."
---

### [BLOCKING] The card fixer still stops on a question

**Location:** internal/gatecli/cardfixer.go:135

**Issue:** The callback is installed on a shuttle runner, but the card fixer never uses that runner's normal wait.
`internal/gatecli/detachfixer.go:54` calls the cold fixer's `Await`.
`internal/gatefixer/fixer.go:35` delegates to `gatefixer.Await`.
That function classifies a turn ending without a reply as `DeadReasonAsking` and stops the strand.
It neither posts the new notice nor waits for a parent answer.
`cardFixerSpec` also omits `PromptInputs.ParentName`, rendering the no-parent directive.
This leaves the very question path the changed notifier claims to close unchanged.
The test lens's notifier coverage finding is consolidated here:
the existing card-fixer test substitutes a fake fixer, and the notifier-wiring table excludes gatecli.

### [BLOCKING] An advisory typo prevents execution

**Location:** internal/loomcli/rownotice.go:37

**Issue:** An unknown subscription returns an error, and `newShed` returns no shed.
`PATTERN-refusal-way-forward` requires guards unrelated to correctness to warn and record rather than halt.
This key adds advisory parent mail; invalid subscriptions do not make producer execution unsafe or incorrect.
The board entry does not require this refusal, and the range contains no departure explaining it.
The focus directive asks whether refusal is implemented; it is, but the governing constraint forbids it.

### [BLOCKING] Tests bypass activation and lifecycle behavior

**Location:** internal/loomcli/rownotice_test.go:55

**Issue:** The tests call `watchRows` and the wrapped producer directly, manually writing their expected history.
Removing the wrapper installation from `newShed` would leave them passing.
The existing wiring test at `internal/loomcli/wiring_test.go:497` calls `loomrecipe.New` directly too.
No test proves that a configured subscription reaches the run or survives its actual lifecycle transitions.
This violates the public-surface testing rule in `PATTERN-test-speed`.
The failure test also calls once, with a producer returning an empty pointer and nil error.
It leaves failed-post deduplication, storage-failure delegation, and populated return values unchecked.
These are boundary and behavior contracts introduced by this diff, not tests of hypothetical internal misuse.
The test lens's first and third findings are consolidated here.

### [MEDIUM] Role restrictions contradict parent routing

**Location:** contracts/stencils/parent/parent-directive-operator-ban.md:4

**Issue:** The shared directive now requires every operator question to go to the parent.
The fixer's opening stencil permits only the parent-ruling flag and forbids other messages about the review.
Both reviewer openings still end with a blanket prohibition on messages.
Thus a tradeoff question can still be blocked by the more specific role instruction.
Part 1 expressly requires closing remaining paths, including perch fixers; changing only the shared line leaves this gap.

## Review evidence

Reviewed commits from `8998a865c8088e5b677a11810386a593a5b3c20e` through `8bbd1196cee10b9bb9529c37f830312577f535bd`.
The decision record and board entry T2 supplied the scope; no amendments or PATTERN declarations were present.
All four lens reports are saved beside this report.
The changed Go packages passed `go build ./internal/loomcli ./internal/gatecli ./internal/loomengine ./internal/shedrun`.
No test suite was rerun during this review; the supplied reach and mutation reports were treated as advisory evidence.

The visit key counts additional folded calls correctly and stays unchanged across yield or cancellation pause.
The wrapper returns the inner producer's outcome, output, and error unchanged.
The key is declared in Config, ConfigSpec, and the template, with CategoryNone exposing the local layer.
The changed docs cover the key and its intended use.
The implementation explicitly makes failed posts non-retryable and storage failures warning-only;
no delivery-guarantee defect is raised merely because advisory mail can fail.
The mutation survivors alone justify neither an internal arithmetic assertion nor a warning-log test.

## Rejected

- The correctness lens's acceptance of the card-fixer integration does not hold beyond runner construction: the actual wait bypasses it.
- The scope lens's acceptance of completed question routing is narrowed by the unchanged cold-fixer and role-ban paths in F1 and F4.
- A separate test of the exact verdict-count constant is unnecessary when positive increments preserve visit identity.
- A separate warning-log assertion is unnecessary for the reported notification-error mutant.
