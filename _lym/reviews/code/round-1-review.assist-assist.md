---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/summaryparser/description.go:124
    summary: A fenced example satisfies Merge danger validation without a real section
    fix: "In mergeDangerSection and its label scan, exclude fenced code from section headings, boundaries and field values. Handle backtick and tilde fences. Add rows to TestValidateDescription_Table for a fenced example alone, an example preceding an invalid real section, and fenced labels within an otherwise empty real section; assert the missing-section or missing-field findings."
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: scope
    location: internal/heddleengine/glossary.md:8
    summary: Glossary definitions include implementation details the entry excludes
    fix: "Remove the Bolt checkout detail from board, the filenames and paths from decision record and change description, and the tmux server detail from strand. Keep the domain meanings and prohibited synonyms. Put any necessary storage or runtime explanation in reference documentation."
    origin: lens:code-scope
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/summaryparser/description.go:139
    summary: No validator row checks that a level-one heading ends Merge danger
    fix: "Add a row to TestValidateDescription_Table with # Notes between ## Merge danger and its Door and Blast radius lines. Expect CheckMergeDangerDoor and CheckMergeDangerBlastRadius, as for the existing level-two boundary row."
    origin: lens:code-tests
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/loomengine/discussiontable.go:184
    summary: Discussion glossary delivery is exercised without checking the delivered text
    fix: "Extend TestDiscussionTable to assert that the rendered chair prompt contains heddleengine.Glossary(). Derive the expected text from that shared source rather than copying its prose, so unrelated nonempty marker content fails while vocabulary edits remain free."
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: design
    location: internal/summaryparser/description.go:152
    summary: The terminal-period normalization has no assertion that detects its removal
    fix: "Add a successful TestValidateDescription_Table row whose Door is one-way. or a backtick-wrapped one-way followed by a period. Keep the assertion on the complete finding list; the existing punctuated radius alone cannot detect removal of the period-trimming statement."
    origin: lens:code-tests
  - id: F6
    severity: LOW
    class: consistency
    location: internal/heddleengine/store_test.go:191
    summary: The new shipped-entry row asserts the glossary heading's static wording
    fix: "Remove the literal glossary heading expectation from TestStore/ShippedEntriesArePinned while preserving the new entry's slug, sole target and order assertions. If heading structure needs coverage, assert its level without fixing its words."
    origin: lens:code-tests
---

### [MEDIUM] Fenced examples satisfy the section check

**Location:** internal/summaryparser/description.go:124

**Issue:** The scan accepts a Merge danger heading and labels inside a Markdown code fence.
A description containing only an example therefore passes without stating its own merge danger.
An earlier example also masks a later real section with missing or invalid fields.
The branch's change description itself contains an example before its final section,
so this ordering is present in an actual producer output.
The final-summary spec requires a section of the description, not an example of that section.

### [MEDIUM] Definitions contain storage and runtime details

**Location:** internal/heddleengine/glossary.md:8

**Issue:** Part 3 asks for a glossary free of implementation detail, with explanations kept in docs.
The board definition names its checkout, the record and description definitions name files,
and the strand definition names its tmux server.
These details exceed the requested vocabulary definitions.

### [MEDIUM] Level-one section boundaries lack coverage

**Location:** internal/summaryparser/description.go:139

**Issue:** The contract ends the section at either a level-one or level-two heading.
The table checks level two and level three, but never level one.
Removing the level-one clause lets labels under a later section satisfy Merge danger.
The reported surviving clause deletion corroborates this missing assertion.

### [MEDIUM] Discussion delivery lacks a value assertion

**Location:** internal/loomengine/discussiontable.go:184

**Issue:** Existing Discussion tests construct and render the prompt but assert other marker values.
Replacing the glossary value with unrelated nonempty text preserves those checks.
The required shared-source delivery needs an assertion on the rendered value, not on the vocabulary's wording.

### [LOW] Period handling lacks an observable check

**Location:** internal/summaryparser/description.go:152

**Issue:** The helper documents terminal-period removal, and the mutation report shows that removing it survives.
The current punctuated radius still counts as one word without normalization.
A punctuated Door distinguishes the intended normalization from the mutant.

### [LOW] A new assertion freezes static prose

**Location:** internal/heddleengine/store_test.go:191

**Issue:** The new row requires the exact heading text through the comparison at lines 207–209.
The rubric prohibits new assertions on static prose.
Slug, target, ordering and heading structure can be checked independently of those words.

## Review evidence

Reviewed the committed range from `8998a865c8088e5b677a11810386a593a5b3c20e` to HEAD using the supplied pieces.
The board entry and parent amendment authorize all change groups; no scribe change is present.
The rubric contains the requested door check, tautological-test rule and mutation-evidence guidance.
Glossary delivery stays scoped to orch and Discussion, with the conflict call-out and settled-term duty present.
The nicknames entry remains in place.
Changed fixtures preserve existing assertions, apart from the additional static wording assertion identified above.

`lym loom validate-description` accepted the current description.
No new test run was used to duplicate the supplied gate results.
The newline survivor in shipped.go is normalized again by passage rendering.
The section-index survivors do not identify a distinct observable defect beyond the findings above.
Invalid compiled-in glossary data is a programmer invariant, not a missing external failure-path test.

All lens findings were retained; none required a rejected-findings entry.
The APPROVED verdict follows the format's severity rule: no finding has BLOCKING severity.
