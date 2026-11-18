---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    origin: lens:code-scope
    location: internal/boardengine/intake.go:205
    summary: Import warns about named evidence but never preserves it as the first goal requires.
    fix: Add import-time preservation of resolvable named evidence for both new notes and folds, embedding its contents in the entry or storing it in a durable referenced destination. Keep an explicit missing-evidence warning when the source cannot be resolved. Add a public import regression that removes the original capture after import and verifies that the imported evidence remains readable.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/selfreportcli/cli.go:209
    summary: Invalid UTF-8 attachments are silently changed by JSON encoding.
    fix: Validate attachment bytes as UTF-8 before converting them to inline text, and refuse invalid input with guidance to supply a UTF-8 textual excerpt. Add a row to the existing refusal test that supplies invalid bytes and verifies that transport is not reached.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/boardengine/intake.go:158
    summary: Native Windows paths under the never-tracked directory receive no evidence warning.
    fix: Recognize both slash and backslash component separators in localPathPattern while preserving original path spelling in LocalPaths. Extend the existing import table with a native Windows capture path and a nonmatching mid-component directory name.
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: plugins/ly/skills/board/SKILL.md:239
    summary: The board skill treats every local path mention as proof that its evidence is absent.
    fix: Say that local_paths identifies references to ephemeral source files, direct readers to inspect the imported body and attached excerpts, and limit the instruction against searching to unavailable original files. Align the import command help with this distinction.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/boardengine/intake.go:138
    summary: The changed fold branch has no test asserting its new evidence-gap result fields.
    fix: Extend TestImportIssueFold with a blank issue body and a comment naming a local path. Assert the title-bearing note, preserved comment and existing body, EmptyBody, and LocalPaths on the returned result.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/selfreportcli/cli_test.go:337
    summary: Attachment limit tests do not cover complete-body accounting or the acceptance boundary.
    fix: Add rows to the existing success and refusal tables with final rendered bodies exactly at the limit and one rune over it. Include multibyte text and body-plus-attachment inputs that each fit individually but exceed the limit after headings and fences are added.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/boardcli/intake_test.go:459
    summary: The omission assertions also accept explicitly present null JSON fields.
    fix: Use the two-result map lookup to assert absence of empty_body and local_paths when no corresponding gap exists; retain value assertions for present fields.
---

### [BLOCKING] Import-time preservation is missing

**Location:** internal/boardengine/intake.go:205

**Issue:** The first Goal bullet requires copying the evidence an imported issue names into a durable destination.
The new code lists paths and appends a warning, then retains only the existing issue body and comments.
Both import destinations use this content builder; neither reads named evidence.
Even a capture still available at import time remains vulnerable to deletion with its source worktree.
Filing-time attachments help future reports but do not implement this import-time requirement.
The board entry and decision record contain no amendment authorizing the substitution.
The landing summary acknowledges the substitution; it does not claim an unrecorded parent correction.

### [MEDIUM] Attachment bytes can change silently

**Location:** internal/selfreportcli/cli.go:209

**Issue:** A file containing byte `0xff` passes the read and becomes a Go string without validation.
The request passes through `selfreportengine.CreateIssue` to go-github's JSON encoder.
JSON encoding replaces invalid UTF-8 with U+FFFD, so successful filing can lose the supplied evidence bytes.
This is an external-input boundary, not an impossible internal state.

### [MEDIUM] Windows evidence paths are missed

**Location:** internal/boardengine/intake.go:158

**Issue:** The matcher requires the literal slash after `.lym` and slash-separated prefixes.
For `C:\work\.lym\shuttle\abc\startup-capture.txt`, it produces no path.
Consequently both the import note and result flags omit the warning for native Windows run evidence.
The failure is shared by new notes and folds.

### [MEDIUM] The skill can direct readers away from preserved evidence

**Location:** plugins/ly/skills/board/SKILL.md:239

**Issue:** Detection examines path mentions, not whether the issue also quotes or attaches their contents.
An issue can name `.lym/shuttle/run/capture.txt` and include the complete capture in an attachment section.
Import preserves that section and still emits `local_paths`.
The skill nevertheless says that evidence is absent and tells its reader not to search for it.
The command help also describes path detection as proof that evidence is not on the board.

### [MEDIUM] Fold result coverage is missing

**Location:** internal/boardengine/intake.go:138

**Issue:** The new-entry test exercises gap detection only with `Slug`.
Existing fold tests pass populated bodies without local paths and assert neither new result field.
Restoring the former direct return from `foldIssue` would drop both flags without failing those assertions.

### [MEDIUM] The body limit boundary is untested

**Location:** internal/selfreportcli/cli_test.go:337

**Issue:** The refusal fixture already exceeds the limit before any heading, fence or body is added.
It cannot distinguish complete-body accounting from checking each attachment independently.
The suite also cannot distinguish rune counting from byte counting or acceptance from refusal at equality.
These are observable parts of the new limit behavior.

### [LOW] Null is accepted where omission is required

**Location:** internal/boardcli/intake_test.go:459

**Issue:** A missing map key and an explicitly null JSON value both produce `nil` on single-result lookup.
Formatting either also produces `<nil>`.
The assertions therefore do not enforce the test's stated omission contract.

## Rejected

- The tests lens's separate close-error coverage finding: success fields are built once before the branch,
  and the existing failure test checks that this shared result reaches the error envelope.
  There is no separate new gap transformation on that path requiring another finding.
- A disagreement between persisted import notes and returned flags: both derive from the same unchanged issue value.
  The board result refresh replaces only `Entry` and preserves the new fields.
- Missing flags on an already-recorded response: this remains a no-op returning recording slugs,
  rather than a fresh import or an assertion about the previously stored entry's completeness.

## Validation and focus coverage

All four requested lenses returned reports, saved verbatim beside this report.
The review covers commits in `9c5bd13101af1686192cd463afa2b7b583b72697..HEAD`.
No existing assertion was loosened, and the new tests add no waits or sleeps.
Fences, nil bodies, empty attachments, stdin composition and read-failure propagation were traced through the code.
The fence is longer than every backtick run in its content, and the limit arithmetic includes rendered framing.
The external GitHub limit was not independently verified.
The changed documentation and stencil preserve the public-content restriction and existing filing-failure behavior.

The attempted targeted tier-0 gate could not start:
its holder record under the hub's `_bolt/.lym/gate` directory was refused by the read-only filesystem.
No test-pass claim is made from that attempt.
