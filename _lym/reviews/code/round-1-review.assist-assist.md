---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/driftengine/judgesnapshot.go:331
    summary: The decision-record exemption hides judge-written additions and carry-over edits without checking their origin.
    fix: Restrict the exemption to content established as an authorized decision-add write and detect subsequent working-tree edits. Preserve carry-over blocks unless an authorized operation changed them. Alternatively, remove the content exemption and refuse decision add while a judge runs, with retry guidance. Extend TestJudgeSnapshot to distinguish parent additions from judge-written additions and carry-over changes, and update the package documentation.
    origin: lens:code-correctness
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/driftengine/expectation.go:297
    summary: The baseline-only refusal prevents expecting the first unkeyed line added to a fully keyed case.
    fix: Remove the baseline-only range rejection from checkCase while retaining the expect-agent instruction to use keys for keyed changes. Update the corresponding documentation and replace RangeOverKeyedCase with acceptance coverage for an added unkeyed line in a previously fully keyed case. Include mixed and empty baseline cases in the existing parser table and assert that the resulting range covers the added unkeyed cell without covering a keyed cell.
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/driftengine/judgesnapshot.go:331
    summary: Parent decision additions that declare or withdraw a PATTERN change still falsely halt the judge.
    fix: Include the PATTERN-section writes of authorized decision add operations in the provenance-aware handling required by F1, or refuse those operations during a judge session with retry guidance. Add cases for both a PATTERN declaration and a withdrawal to the existing snapshot behavior test; do not broadly exempt hand edits to the PATTERN section.
    origin: lens:code-affected-set
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/driftengine/judgesnapshot_test.go:324
    summary: The decision-record exemption lacks coverage for missing copies, read errors, and malformed appended headings.
    fix: Extend TestJudgeSnapshot with cases for a missing decision copy, an unreadable copy, an unreadable current record, and a malformed appended heading. Assert that missing or malformed evidence cannot exempt a changed record and that read failures return errors. Use the existing fake checkouts and permkit for permission failures; adapt these cases to the final exemption design rather than testing private helpers.
    origin: lens:code-tests
---

### [BLOCKING] Judge edits bypass the record guard

**Location:** internal/driftengine/judgesnapshot.go:331

**Issue:** The exemption checks content shape without establishing who wrote it or whether it was committed.
Take a snapshot of the original record, append a valid added-decision entry directly, and leave HEAD unchanged.
`IdentityText` strips the entry, and `CheckAddedEntriesSubset(now, before)` finds no missing prior entry.
The record path is then exempt even when it appears in the checkout's changed paths.
`IdentityText` also strips well-formed carry-over blocks, hiding their alteration or removal.
The relevant operations are in `internal/discussionparser/identity.go:19` and `judgesnapshot.go:270`.
This weakens the guard over the judge's governing record beyond the Goal's exclusion of decision-add commits.
Documenting the weakness in `internal/driftengine/doc.go:279` does not authorize it.

### [BLOCKING] A valid unkeyed addition becomes inexpressible

**Location:** internal/driftengine/expectation.go:297

**Issue:** A fully keyed baseline does not imply that the current capture has no unkeyed lines.
With key pattern `^(?P<key>[^=]+)=(?P<value>.*)$`, take baseline `k=1` and append an unkeyed `plain` line.
`internal/driftengine/diff.go:225` assigns the new line position 1;
its diff loop at line 312 emits an added-line cell with that identity.
A range of `1` covers it through `Expectation.Cover`, but the new parser condition rejects that expectation.
A key entry cannot cover the unkeyed cell.
The refusal also reaches amendments through `internal/driftengine/amend.go:156`.
The prompt clarification already implements the Goal's permitted key-based instruction without this regression.

### [MEDIUM] Supported parent additions remain falsely attributed

**Location:** internal/driftengine/judgesnapshot.go:331

**Issue:** The same parent decision-add command supports PATTERN declarations and withdrawals.
`internal/loomcli/decision.go:999` constructs those additions;
`internal/discussionparser/append.go:127` also changes their PATTERN-section bullets.
`IdentityText` leaves those bullets in the compared text, so the legitimate commit receives no exemption.
The judge is still halted for this supported variant of the operation the Goal asks to accommodate.
This is separate from F1: identifying an authorized writer must also account for the operation's complete output.

### [MEDIUM] Failure boundaries have no regression coverage

**Location:** internal/driftengine/judgesnapshot_test.go:324

**Issue:** The added rows test a successful addition, an edit elsewhere, and a dropped existing entry.
None tests the new copy's absence, read failures, or malformed appended headings.
The existing failure case at line 442 concerns untracked-file copying, not the decision-record comparison.
The new code currently returns no exemption for absent copies and propagates read errors;
no snapshot assertion protects those fail-closed outcomes.

## Consolidation

The scope lens's exemption finding is merged into F1.
The affected-set lens's unkeyed-addition finding is merged into F2.
Its parent-decision variant finding and the correctness lens's equivalent finding are merged into F3.
The mixed and empty baseline coverage finding is included in F2's parser correction.
F3 uses MEDIUM because it leaves a supported variant of the original false halt unresolved.
The new wiring reaches the sole production constructor and survives recipe dependency copying.
Other changed files remain subject to the ordinary snapshot comparison.
No separate finding concerns the static expect-agent wording, test timing, or redundant tests.

## Validation

The review covers commits after `0acf880f6bb388d1e7664a1dd42916ae09c1c05b` through HEAD.
The board task and decision-record Goal agree, and no scope amendments or PATTERN departures were present.
All lens reports were saved before consolidation.
The attempted `lym gate test ./internal/driftengine --tier 0` could not acquire a gate slot:
the hub's `_bolt/.lym/gate` directory is read-only in this sandbox.
No tests ran in that attempt; the findings above rest on source traces and concrete inputs, not a claimed test failure.
